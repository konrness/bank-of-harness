# Demo Guide

This document describes how to develop and add features to the Bank of Anthos application in your local environment to demonstrate Harness Software Delivery Platform capabilities.

## Prerequisites 

1. [Docker Desktop](https://www.docker.com/products/docker-desktop) with Kubernetes Cluster running
1. [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) (can be installed separately or via [gcloud](https://cloud.google.com/sdk/install)) 
1. [skaffold **1.27+**](https://skaffold.dev/docs/install/) (latest version recommended)
1. [OpenJDK **17**](https://openjdk.java.net/projects/jdk/17/) (newer versions not tested)
1. [Maven **3.8**](https://downloads.apache.org/maven/maven-3/) (newer versions not tested)
1. [Python **3.7+**](https://www.python.org/downloads/)  
1. [piptools](https://pypi.org/project/pip-tools/)

### Installing JDK 17, Maven 3.8 and Skaffold on MacOS

Install OpenJDK 17:
```
brew install openjdk@17
sudo ln -sfn /usr/local/opt/openjdk@17/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-17.jdk
java -version
```

Install Maven 3.8:
```
brew install maven
mvn -version
```

Install Skaffold:
```
brew install skaffold
```


## Testing your changes locally 

We recommend you test and build directly on Kubernetes, from your local environment.  This is because there are seven services and for the app to fully function, all the services need to be running. All the services have dependencies, environment variables, and secrets and that are built into the Kubernetes environment / manifests, so testing directly on Kubernetes is the fastest way to see your code changes in action.

You can use the `skaffold` tool to build and deploy your code to the GKE cluster in your project. 

Make sure that you export `PROJECT_ID` as an environment variable (or add to your `.bashrc` before running either of these commands)

### Option 1 - Build and deploy continuously 

The [`skaffold dev`](https://skaffold.dev/docs/references/cli/#skaffold-dev) command watches your local code, and continuously builds and deploys container images to your Docker Desktop cluster anytime you save a file. Skaffold uses Docker Desktop to build the Python images, then [Jib](https://github.com/GoogleContainerTools/jib#jib) (installed via Maven) to build the Java images. 

```
skaffold dev
```

### Option 2 - Build and deploy once 

The [`skaffold run`](https://skaffold.dev/docs/references/cli/#skaffold-run) command build and deploys the services to your Docker Desktop cluster one time, then exits. 

```
skaffold run
```


## Adding External Packages 

### Python 

If you're adding a new feature that requires a new external Python package in one or more services (`frontend`, `contacts`, `userservice`), you must regenerate the `requirements.txt` file using `piptools`. This is what the Python Dockerfiles use to install external packages inside the containers.

To add a package: 

1. Add the package name to `requirements.in` within the `src/<service>` directory:

2. From inside that directory, run: 

```
python3 -m pip install pip-tools
python3 -m piptools compile --output-file=requirements.txt requirements.in
```

3. Re-run `skaffold dev` or `skaffold run` to trigger a Docker build using the updated `requirements.txt`.  


### Java 

If you're adding a new feature to one or more of the Java services (`ledgerwriter`, `transactionhistory`, `balancereader`) and require a new third-party package, do the following:  

1. Add the package to the `pom.xml` file in the `src/<service>` directory, under `<dependencies>`. You can find specific package info in [Maven Central](https://search.maven.org/) ([example](https://search.maven.org/artifact/org.postgresql/postgresql/42.2.16.jre7/jar)). Example: 

```
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
        </dependency>
```


2. Re-run `skaffold dev` or `skaffold run` to trigger a Jib container build using Maven and the updated pom file. 


### Deploying to GKE Cluster

```
export PROJECT_ID=[myprojectid]

skaffold build --file-output=deploy/artifacts.json --default-repo=gcr.io/${PROJECT_ID}/bank-of-harness --push=true
skaffold render --build-artifacts=deploy/artifacts.json --output=deploy/manifests.yaml --namespace=bank-of-harness
kubectl apply -f deploy/manifests.yaml
```
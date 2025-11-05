# Copyright 2021 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Use the official Python docker container, slim version, running Debian
FROM cgr.dev/chainguard/python:latest

LABEL org.opencontainers.image.base.name="cgr.dev/chainguard/python"
LABEL org.opencontainers.image.base.tag="latest"
LABEL org.opencontainers.image.base.digest="sha256:2a8c7d9efdae9b636de80a187a4ce31374ffb3883a7be06232d25a96e8538bf4"

# Set our working directory
WORKDIR /app

# show python logs as they occur
ENV PYTHONUNBUFFERED=0

# explicitly set a fallback log level in case no log level is defined by Kubernetes
ENV LOG_LEVEL info

# Install dependencies.
COPY requirements.txt .
RUN pip install -r requirements.txt

# Add application code.
COPY . .

# Start server using gunicorn
CMD gunicorn -b :$PORT --threads 4 --log-config logging.conf --log-level=$LOG_LEVEL "frontend:create_app()"

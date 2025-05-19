# Sundial Trino Build and Deployment Guide

This document provides instructions for building Trino and deploying it to Sundial's ECR registry.

## Prerequisites

- AWS CLI configured with appropriate permissions
- Docker installed and running

## Environment Setup

### Install SDKMAN and Java 23

```bash
# Install SDKMAN
curl -s "https://get.sdkman.io" | bash

# Install Java 23 from Temurin
sdk install java 23-tem

# Initialize SDKMAN in the current shell
source "$HOME/.sdkman/bin/sdkman-init.sh"
```

## Building Trino

Set up Java environment and build Trino:

```bash
# Configure Java environment
export JAVA_HOME="$HOME/.sdkman/candidates/java/23-tem"
export PATH="$JAVA_HOME/bin:$PATH"

# Build Trino (skip tests for faster build)
./mvnw clean install -DskipTests=true -e
```

## Docker Image Build and Push

Build and push the Trino Docker image to Sundial's ECR registry:

```bash
# Set AWS region
region=us-east-2

# Get AWS account ID dynamically from your AWS credentials
aws_account_id=$(aws sts get-caller-identity --query Account --output text)
echo "Using AWS Account ID: ${aws_account_id}"

# Log in to ECR
aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${aws_account_id}.dkr.ecr.${region}.amazonaws.com

# Navigate to the Docker directory
cd core/docker

# Build the Docker image for AMD64 architecture
./build.sh -a amd64

# Tag the image (replace perf_test_tag with appropriate tag)
tag=perf_test_tag
docker tag trino:469-amd64 ${aws_account_id}.dkr.ecr.${region}.amazonaws.com/sundial-table-query-engine:${tag}

# Push the image to ECR
docker push ${aws_account_id}.dkr.ecr.${region}.amazonaws.com/sundial-table-query-engine:${tag}
```

## Notes

- Make sure to replace `perf_test_tag` with an appropriate tag for your build
- The Docker image is built for AMD64 architecture; modify as needed for other architectures

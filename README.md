# Flawfinder Code Security Analysis Integration

## Overview

This repository contains the implementation of an automated security analysis process using Flawfinder, a powerful open-source code review tool, designed to enhance code security for C/C++ projects. The integration is designed to seamlessly fit into a Continuous Integration/Continuous Deployment (CI/CD) pipeline, ensuring code quality and security with minimal effort. 

## Features

- **Efficient Code Review**: Flawfinder is used to identify common security pitfalls and vulnerabilities in C/C++ code, enhancing code quality and security.
- **Automated Process**: Integration with GitLab and Github Actions automates code security analysis, making it an integral part of your development workflow.
- **Transparency and Accessibility**: Results of code analysis are transparently displayed within Github Actions, facilitating early issue resolution.
- **Reproducibility**: Docker containers ensure consistent and reproducible code analysis across different environments.
 
## Getting Started

Follow these steps to integrate Flawfinder into your project:

1. **Install Docker**: Ensure that Docker is installed on your local development environment. You can download and install Docker from [here](https://www.docker.com/get-started).

2. **Add GitHub Secrets**:
   - **DOCKER_USERNAME** and **DOCKER_PASSWORD**: In your GitHub repository, navigate to "Settings" > "Secrets" and add your Docker Hub credentials as secrets. Use `DOCKER_USERNAME` for your Docker Hub username and `DOCKER_PASSWORD` for your Docker Hub password.
   - **GITLAB_TOKEN**: Create a personal access token in GitLab with appropriate permissions for Docker registry access. In your GitHub repository, navigate to "Settings" > "Secrets" and add your GitLab access token as a secret with the name `GITLAB_TOKEN`.

3. **Customize GitHub Action**:
   - Create a GitHub Actions workflow file (e.g., `.github/workflows/flawfinder.yml`) in your repository with the following content:

   ```yaml
   name: Flawfinder Code Analyzer

   on:
     push:
       branches:
         - main

   jobs:
     flawfinder:
       runs-on: ubuntu-latest

       steps:
         - name: Checkout repository
           uses: actions/checkout@v2

         - name: Run Flawfinder and capture output
           env:
             DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
             DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
             GITLAB_TOKEN: ${{ secrets.GITLAB_TOKEN }}
           run: |
             echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
             echo "$GITLAB_TOKEN" | docker login registry.gitlab.com --username tipu.solehria --password-stdin
             docker run \
               --rm \
               --volume "$PWD":/tmp/app \
               --env CI_PROJECT_DIR=/tmp/app \
               --env SECURE_LOG_LEVEL=debug \
               -w /tmp/app \
               registry.gitlab.com/gitlab-org/security-products/analyzers/flawfinder:latest /analyzer run > report.txt 2>&1

             # Remove color escape codes
             sed -i 's/\x1b\[[0-9;]*m//g' report.txt

         - name: Display Flawfinder Report
           run: cat report.txt

         - name: Upload report
           uses: actions/upload-artifact@v2
           with:
             name: flawfinder-report
             path: report.txt

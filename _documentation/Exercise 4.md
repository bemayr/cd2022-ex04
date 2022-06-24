# Exercise 4

## References
- Repository: https://github.com/bemayr/cd2022-ex04
- This documentation file: https://github.com/bemayr/cd2022-ex04/blob/main/_documentation/Exercise%204.md

## Local Run

The following image shows that building and running the container locally worked seamlessly.
![GitHub Actions Successful Run](./Exercise%204.1.png)

## GitHub Actions

The following GitHub Actions file is my solution for the given exercise. It was quite some work to get Trivy to work in such a way that it was given. Now I run the Trivy vulnerability scanner thrice, once for the Dockerfile, a second time for the repositorys source files and a third time for the infrastructure code. The Image gets tagged with a short version of the commit sha and pushed only if it builds correctly.

```yml
name: Build and Push Docker Image

on:
  push:
    branches: [master]
  pull_request:

jobs:
  test-build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout 🛎️
      uses: actions/checkout@v3

    - name: Declare the Short Git Commit Hash #️⃣
      id: vars
      shell: bash
      run: echo "::set-output name=sha_short::$(git rev-parse --short HEAD)"

    - name: Build an image from Dockerfile 🐋
      run: docker build -t bemayr/demo:${{ steps.vars.outputs.sha_short }} .

    - name: Run Trivy vulnerability scanner for the Docker Image 🔍[image]
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: bemayr/demo:${{ steps.vars.outputs.sha_short }}
        format: 'sarif'
        output: 'trivy-results-image.sarif'
        exit-code: '1'
        vuln-type: 'os,library'
        severity: 'CRITICAL,HIGH'

    - name: Upload Trivy scan results to GitHub Security tab 📄[image]
      uses: github/codeql-action/upload-sarif@v2
      if: always()
      with:
        sarif_file: 'trivy-results-image.sarif'
        category: "image"

    - name: Run Trivy vulnerability scanner for the Repository 🔍[repository]
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        ignore-unfixed: true
        format: 'sarif'
        output: 'trivy-results-repository.sarif'
        exit-code: '1'
        severity: 'CRITICAL'

    - name: Upload Trivy scan results to GitHub Security tab 📄[repository]
      uses: github/codeql-action/upload-sarif@v2
      if: always()
      with:
        sarif_file: 'trivy-results-repository.sarif'
        category: "repository"

    - name: Run Trivy vulnerability scanner for IaC Files 🔍[infrastructure]
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'config'
        hide-progress: false
        format: 'sarif'
        output: 'trivy-results-infrastructure.sarif'
        exit-code: '1'
        ignore-unfixed: true
        severity: 'CRITICAL, HIGH'

    - name: Upload Trivy scan results to GitHub Security tab 📄[infrastructure]
      uses: github/codeql-action/upload-sarif@v2
      if: always()
      with:
        sarif_file: 'trivy-results-infrastructure.sarif'
        category: "infrastucture"

    - name: Login to DockerHub 🔐
      uses: docker/login-action@v2
      if: github.ref == 'refs/heads/master'
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: Push built Docker Image ⤴️
      uses: docker/build-push-action@v2
      if: github.ref == 'refs/heads/master'
      with:
        push: true
        tags: bemayr/demo:latest, bemayr/demo:${{ steps.vars.outputs.sha_short }}
```
Some example runs of the GitHub Actions workflow above are visible beneath. The most interesting part is the (1) that is highlighted.

![GitHub Actions Successful Run](./Exercise%204.3.png)

Instead of outputting the result of the vulnerability scanning in the console of the Action, I create `sarif`-files, which can be integrated in GitHub's Security Tab as shown in the next two screenshots. This is quite a neat feature.

![GitHub Actions Successful Run](./Exercise%204.4.png)
![GitHub Actions Successful Run](./Exercise%204.5.png)

## Docker Hub Deployment

The following screenshot shows that the Deployment to Dockerhub worked perfectly. Instead of specifying a password, I created a secret token. The Docker Hub URL is: https://hub.docker.com/u/bemayr

![GitHub Actions Successful Run](./Exercise%204.2.png)

## This is an example of github action with docker


```python
name: CI/CD for Dockerized Flask App

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
## Build the docker
  dockerbuild:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Build The Docker Image
      run: docker build . --file DockerFile --tag workflow-test:$(date +%s)

## Build and test the docker
  build-and-test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.9'

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install flask
        pip install pytest

    - name: Run tests
      run: |
        pytest

## Build and publish the docker
  build-and-publish:
    needs: build-and-test
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Login to DockerHub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        file: ./DockerFile
        push: true
        tags: ${{ secrets.DOCKER_USERNAME }}/flasktest-app:latest

    - name: Image digest
      run: echo ${{ steps.build-and-publish.outputs.digest }}

```


## Create docker password through token
- dockerhub
    - account settings
        - Personal access token
            - Generate new token

#### After generating a token
- Go to Github
    - Settings
      - Secrets and variables
          - Actions
              - New repository secret for "DOCKER_USERNAME"
              - New repository secret for "DOCKER_PASSWORD"

- Push all the code to github -> go to Actions to see the deployment


## You can see the docker image in the docker dektop
- If not seen the run in a cmd-prompt below code
```python
docker pull mannshrestha10656/flasktest-app
```

- To run a docker image locally
```python
docker run -p 5000:5000 mannshrestha10656/flasktest-app:latest
```

- Go to web browser
  - localhost:5000/

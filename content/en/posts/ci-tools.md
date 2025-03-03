---
author: alex
category:
  - devnet
  - study-material
date: "2024-01-18T19:46:23+00:00"
guid: https://netwithalex.blog/?p=103
title: CI Tools
url: /ci-tools/
draft: true
---
## Describe characteristics and concepts of build /deploy tools such as Jenkins, Drone, or Travis CI

In the [last post](/devops-ci-cd-pipeline/) we talked about the concepts of CI/CD, emphasizing the importance of the **_build_** phase as part of Continuous Integration process. It is now time to dig deeper on the following CI tools: Jenkins, Drone, Travis and GitLab.

### Jenkins

Jenkins is a **_free_** automation server (open source) designed to automate parts of the software development process, most notably the building, testing, and deployment of code changes. It is Java based, has a large community and has been out there for a while.

Offering modeLanguage/StructureRootURLVM/Container(self-managed)GroovyJenkinsfilehttps://www.jenkins.io/


Its pipeline is maintained in Groovy format and processed as Groovy script. This is the file Jenkins will look for by default

Here is an example to understand the structure:

```
pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        <...>
      }
    }

    stage('Build') {
      steps {
        <...>
      }
    }

    stage('Test') {
      steps {
        <...>
      }
    }

    stage('Deploy') {
      steps {
        <...>
      }
    }
  }

  post{
    <...>
  }
}
```

- ` ``stages` ` ` **`-`** 4 (checkout, build, test and deploy)
- ` steps ` **`-`** actions to execute on each of the stages
- ` post ` `-` Sections containing actions to execute based on the result of the pipeline (success, failure, always)

Visit this page to learn more about the [Pipeline](https://www.jenkins.io/doc/book/pipeline/)

Jenkins also offers a User Interface for Pipeline visualization called _[Blue Ocean](https://www.jenkins.io/doc/book/blueocean/)_

![](/wp-content/uploads/2024/01/Screenshot-2024-01-31-at-07.26.51.png)https://www.jenkins.io/blog/2016/05/26/introducing-blue-ocean/

The idea behind it is to improve the developer experience - make it easier to understand the Pipeline, make changes and monitor status and issues arise.

* * *

### Drone

Built on a container technology. Feature set can be augmented through **plug-ins** that can work with source control technologies (GitHub, Bitbucket, GitLab, etc).

Two versions available - **Free** for Open Source and cloud service (not free).

Offering modeLanguage/StructureRootURLSaaS/Container(self-managed)YAML.drone.ymlhttps://www.drone.io/

It is written in Go and uses the following .drone.yml structure:

```
kind: pipeline
name: example

steps:
  - name: install
    image: node:14
    commands:
      - npm install

  - name: test
    image: node:14
    commands:
      - npm test

  - name: deploy
    image: alpine
    commands:
      - echo "Deploying the application"
      # Include deployment commands or scripts here

```

- ` kind ` **`-`** indicate this is a pipeline config (other values could be docker, kubernetes, secret)
- ` name  -` name of the pipeline
- ` steps ` **`-`** indicate individual sections of the pipeline. Each step has its own name, Docker image and commands to execute.

Drones GUI will allow you

Check out this [video](https://youtu.be/kZmOCLCpvmk) by Harness to get familiar with Drone and the GUI.

Visit [Drone.io](https://www.drone.io/) to learn more about this tool, plug-ins and to check the documentation.

* * *

### Travis CI

Integrates very well with Github.com API. Offered as Software-as-a-Service and Open Source projects can be tested for **_free_**. VM/Container options are available as well.

Offering modeLanguage/StructureRootURLSaaS/Container(self-managed)YAML.travis.ymlhttps://travis-ci.org

```
---
language: node_js
node_js:
  - "14"

env:
  - CI_REG_IMG_APP="my_node_app"

before_script:
  - echo "Logging into DockerHub..."
  - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

script:
  - echo "Building Docker image..."
  - docker build -t $CI_REG_IMG_APP .

deploy:
  <...>

```

- ` language ` and ` node  ` **`-`** Defines language to be used and versions
- ` env ` **`-`** Environment variable definition happens here
- ` before_script ` **`-`** Contains commands to be executed before running the main build script
- ` script  ` **`-`** Contains the main build script
- ` deploy ` **`-`** Specifies deployment configuration

I'd recommend checking out this [video](https://www.youtube.com/watch?v=_Og2kydTLWk) that shows how easy it is to integrate with Github and getting started with Travis.

* * *

### GitLab

Available Enterprise and Community edition.

Gitlab has many features for DevOps:

- Git Repository
- Docker Image Registry
- Project Board
- CI/CD Pipelines
- Runners

Offering modeLanguage/StructureRootURLSaaS/Container(self-managed)YAML.gitlab-ci.ymlhttps://gitlab.com

```
---
stages:
  - build
  - test
  - deploy

variables:
  NODE_VERSION: "14"

before_script:
  - npm install -g yarn
  - yarn install

build:
  stage: build
  script:
    - yarn build

test:
  stage: test
  script:
    - yarn test

deploy:
  stage: deploy
  script:
    - echo "Deploying application..."
    # Add deployment commands here
  only:
    - main
```

- ` stages ` `-` Defines stages of the pipeline
- ` variables  - ` Defines global variables for the pipeline.
- ` before-script  - ` Specifies commands to be executed before running any job in the pipeline
- ` build  -` Stage to build the application. It executes the `yarn build` command.
- ` test ` `-` Stage to run automated tests before deployment
- ` deploy -` Responsible for deploying the application

Here is an example of how this could be represented graphically.

![](/wp-content/uploads/2024/02/GitLab1.png)

I would suggest exploring some of the [tutorials](https://docs.gitlab.com/ee/tutorials/) to get familiar with GitLab.

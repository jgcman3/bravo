# Table of Contents

- [Table of Contents](#table-of-contents)
- [Welcome to the Bravops!](#welcome-to-the-bravops)
- [Goal](#goal)
- [Concept](#concept)
- [Architecture](#architecture)
- [Block Diagram](#block-diagram)
- [Work Flow](#work-flow)
  - [1. Prepare Resources](#1-prepare-resources)
  - [2. Configure](#2-configure)
  - [3. Initialize](#3-initialize)
  - [4. Synthesize](#4-synthesize)
  - [5. Deploy](#5-deploy)
  - [6. Destroy](#6-destroy)

<br>

# Welcome to the Bravops!

Bravops is an abbreviation for 'Bravo Operations'.<br>
This is the cdk repository of Bravo operations project. All cdk codes are written in typescript. <br>
You can easily install the synthesize, deploy stacks to the cloud server and delete it using script using aws-cdk. <br>
Let's enjoy cloud world ;-)

<br>

# Goal

- <b>Initialize automatically</b>
  - Install packages ( jq, pip3, aws, zip, docker, node modules )
  - Unzip the resources ( lambda, application .. etc )
  - Set the aws account profile
- <b>Synthesize automatically</b>
  - Generate the template files of stacks
- <b>Deploy automatically</b>
  - Deploy stacks to the AWS
- <b>Destroy automatically</b>
  - Destroy stacks from the AWS

<br>

# Concept

![concepts](https://github.com/jgcman3/bravops/blob/images/bravops-concepts.png?raw=true)

<br>

# Architecture

![architecture](https://github.com/jgcman3/bravops/blob/images/bravops-architecture.png?raw=true)

<br>

# Block Diagram

![blockdiagram](https://github.com/jgcman3/bravops/blob/images/bravops-blockdiagram.png?raw=true)

<br>

# Work Flow

<br>

## 1. Prepare Resources

We need some package resources to build our infra. Currently available resources are as follows. <br>

- <b>lambda package</b>
- <b>application package</b>

There are some rules for package resource files.<br>

- <b>All resource packages should be placed in the <b>`./resources`</b> folder.</b>
- <b>All resource packages must be a zip file.</b>
- File format: release_{project}_{package}_{git tag}.zip
- Example: release_{project}_lambda_v2021.01.0016.zip

The elements that should be included in each package are as follows.<br>

- <b>Lambda</b>

  - <b>release.txt :</b> Package released information such as git tag
  - <b>functions :</b> Folder containing the function zip files to be deployed
  - <b>layers :</b> Folder containg the layer zip files to be deployed

- <b>Application</b>

  - <b>release.txt :</b> Package released information such as git tag
  - <b>application.jar :</b> Application file
  - <b>Dockerfile : </b> Dockerfile for creating docker images

<br>

## 2. Configure

Firstly, you should review and modify the configuration of the project.<br>
See the configuration folder. The configuration folder path is <b>`./zconfig`</b>. <br>
There are three configure files.<br>

- <b>`dev.json`</b> : This is for the dev environment.
- <b>`stage.json`</b> : This is for the stage environment.
- <b>`prod.json`</b> : This is for the prod environment.

The below json is a sample configuration format. <br>
The name of the profile must match with your profile and region of aws credentials(~/.aws/config or ~/.aws/credentials)

```json
{
  "account": {
    "id": "123456789112",
    "profile": "aws-bravo-dev",
    "region": "ap-northeast-2"
  },
  "project": {
    "name": "bravo",
    "env": "dev",
    "version": "v0.1.3",
    "description": "The Dev aws infra of the Bravo project.",
    "tags": [
      { "key": "", "value": "" },
      { "key": "", "value": "" },
      { "key": "", "value": "" }
    ],
    "stacks": {
      "core": [
        "vpc", 
        "secrets", 
        "rds", 
        "rds-sub", 
        "cognito", 
        "nlb",
        "..."
      ],
      "service": [
        "kinesis",
        "sns",
        "cloudwatch",
        "s3",
        "ecs",
        "iot",
        "api",
        "..."
      ]
    },
    "debug": true
  },
  "api": {
    "domain": ""
  },
  "rds": {
    "name": "BRAVO_MAIN",
    "port": "3306",
    "username": "admin",
    "tz": "Asia/Seoul",
    "charset": "utf8mb4"
  },
  "sns": {
    "firebase": {
      "apikey": ""
    }
  },
  "iot": {
    "endpoint": ""
  },
  "acm": {
    "apiCertificateArn": ""
  },
  "cloudwatch": {
    "alarmSlackHookUrl": ""
  },
  "resources": {
    "application": {
      "path": "resources/docker",
      "file": "release_bravo_application_v2021.01.0004.zip",
      "out": "application"
    },
    "lambda": {
      "path": "resources/lambda",
      "file": "release_bravo_lambda_v2021.01.0016.zip",
      "out": "lambda"
    }
  }
}
```

<br>

## 3. Initialize

Please run <b>`./scripts/initialize.sh { name of env }`</b> script.

- <b>`initialize.sh`</b> : The script to initialize the project

It initializes the environment by setting the aws profile of the project. <br>
The name of the aws profile that needs to be set is the value set in the project configuration file. <br>
And It installs the necessary packages. The installed packages are as follows. <br>

- <b>Packages:</b> jq, zip, pip3, aws-cli, cdk, docker, npm modules
  - jq is a lightweight and flexible command-line JSON processor
  - zip is a command-line to compresse one or more files or deirectories
  - pip3 is the package installer for Python
  - aws-cli is AWS command-line interface
  - cdk is AWS cloud development kit
  - docker is Open source software container platform
  - node modules are npm packages needed by the project

Plus, If your environment is <b>Window Subsystem Linux</b>, there are things you have to do yourself.

- <b>To do list</b>
  - Install docker desktop on your windows
  - Start the docker desktop program
  - Enable <b>'Expose daemon on tcp://localhost:2357 without TLS'</b> options

<br>

## 4. Synthesize

Please run <b>`./scripts/synthesize.sh { name of env }`</b> script.

- <b>`synthesize.sh`</b> : This is the script to Synthesize the CDK App of the project.

It synthesize the CDK App of the project.<br>
The cloudformation template files are generated to the <b>`out/{env}`</b> folder automatically. <br>
You can check the generated template files in that folder. <br>
If you want to synthesize the specific stack, Please run <b>`-s { the name of stack }`</b> option.

<br>

## 5. Deploy

Please run <b>`./scripts/deploy.sh { name of env }`</b> script.

- <b>`deploy.sh`</b> : This is the script to deploy the CDK App of the project.

It deploy the CDK App of the project to the AWS.<br>
If you want to deploy the specific stack, Please run <b>`-s { the name of stack }`</b> option.

<br>

## 6. Destroy

Please run <b>`./scripts/destroy.sh { name of env }`</b> script.

- <b>`destroy.sh`</b> : This is the script to destroy the CDK App of the project.

It destroy the CDK App of the project from the AWS.<br>
If you want to destroy the specific stack, Please run <b>`-s { the name of stack }`</b> option.

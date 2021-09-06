# CI/CI Pipelines with Gitlab

Notes written after following
Udemy course by Valentin Despa
GitLab CI: Pipelines, CI/CD and DevOps for Beginners.

A PDF of the course note created by the instructor can be downloaded from
[gitlab-ci-course-notes.pdf](https://buildmedia.readthedocs.org/media/pdf/gitlab-ci-course-notes/latest/)

Mikel Sagardia, 2021.
No warranties.

Overview:
1. Introduction
2. Basic CI/CD Workflow with Gitlab
3. Gitlab CI Fundamentals
4. YAML Basics
5. Deploying an Application

## Section 1: Introduction

As I understand, **Continuous Integration and Continuous Delivery (CI/CD)** consists in:
- Frequently merging new features to our versioned code
- Automatically and frequently building our codebase
- Assessing the quality of our code
- Automatically testing our built binaries
- Frequently deploying our packages to production; deployment encompasses: installation in an environment and tests

Therefore, CI/CD has these advantages:
- It ensures that changes are releasable
- It reduces the risk of a new deployment
- It delivers value much faster

Gitlab CI is gaining markets share in contrast to older CI/CD frameworks (e.g., Jenkins) because it integrates all steps for CI/CD in a platform.

Gitlab can be used
- online: repositories are hosted in Gitlab's servers; offers 30h of pipeline time in its free tier. If we want more, we need to pay.
- self-managed: we install the Gitlab platform; we can do unlimited CI/CD in the free tier. But we get no support for it.

### 1.1 Introductory example: `build a car`

The introduction uses the example of building a car:
a `car.txt` file will be created and the likes `chassis`, `engine`, `wheels` added to it.
We log in to Gitlab & create a new private project: `cicd_tests`.
We create a new file with a specific name in the **highest or root** level: `.gitlab-ci.yml`:


```yml
build the car:
  script:
    - mkdir build
    - cd build
    - touch car.txt
    - echo "chassis" >> car.txt
    - echo "engine" >> car.txt
    - echo "wheels" >> car.txt
```

Right vertical menu: CI/CD, Pipelines:
- We'll see the pipeline which is executed
- If we click on the job, we'll see the terminal execution of the pipelines.
- We have currently one job, `build the car`, but we can add more jobs.
- Each job conatains at least one script.
- Note that pipelines do not locally on our host, but on Gitlab runners that are executed on the Gitlab servers; also, any files we create in a job are not saved to our repository.
- Note that 2 spaces must be used for indenting the YAML files.

We can improve that pipeline adding another job called `test the car` as follows:
- We define the job `test the car`, where the `car.txt` file is checked.
- We define `stages` and assign each job a stage so that the order of job execution can be controlled.
- If we don't assign a stage to a job, Gitlab automatically assigns the job the stage `test`
- The order of the stage defines the order of execution of the jobs.
- Note that each job is executed independently and the files produced in them are not saved unless we actively define `artifacts`.
- The `test the car` job needs to check the content of the `car.txt` file created in the job `build the car` with `test` and `grep` Linux commands; therefore, we need to define `artifacts`and add the files/paths we'd like to save.
- If we go to the vertical menu CI/CD > Jobs
    - we see the history of executed jobs
    - we can open the shell of the job clicking on passed/failed
    - we can download the artifacts of a job clicking on the donwload icon

```yml
stages:
  - build
  - test

build the car:
  stage: build
  script:
    - mkdir build
    - cd build
    - touch car.txt
    - echo "chassis" >> car.txt
    - echo "engine" >> car.txt
    - echo "wheels" >> car.txt
  artifacts:
    paths:
      - build/
test the car:
  stage: test
  script:
    - ls
    - test -f build/car.txt
    - cd build
    - cat car.txt
    - grep "chassis" car.txt
    - grep "engine" car.txt
    - grep "wheels" car.txt 
```

### 1.2 Gitlab Architecture

We have two main elements:
1. The **Gitlab server**: it offers the functionality for creating repositories and it saves them in databases. It also controls the pipelines we create, but it delegates their execution to the Gitlab runners.
2. The **Gitlab runners**: docker containers that run the jobs/tasks we define in the pipeline; the Gitlab runners are launched and managed by the Gitlab server. Each runner is also in charge of saving the artifacts. We can also scale up/down the number of runners used for the jobs.

If we look at the job shells, we'll see they are docker images which are built and run.
Within the container, our repository is cloned and the pipeline executed. Then, the artifacts are saved and the container is destroyed.

On the vertical menu under Settings > CI/CD > Runners we can select if we use the default shared runners provided by Gitlab or if we want to use our specific runners; the latter makes sense if we want to perform intensive CPU/GPU work in our pipelines.

## Section 2: Basic CI/CD Workflow with Gitlab

In this section a static website is built using `Node.js` and `npm`.
`Node.js` is a Javascript runtime environment able to run Javascript without needing a browser; `npm` is the Node Package Manager for it.

Installation and Setup (on Mac):
```bash
brew install node
node -v # we should see the version
npm -v # we should see the version
# install Gatsby CLI
npm install -g gatsby-cli
# create a new website repository locally
cd ~/git_repositories
gatsby new static-website
cd static-website
# we start the local developer server
gatsby develop
# now, we can open our website
# on the browser: localhost:8000
#
# On Gitlab, we create a new (private) project
# my-static-website
# and we connect it to the local repository that is created
# when creating the website
# note the name of the branch is master, not main...
git remote add origin git@gitlab.com:mxagar/my-static-website.git
git push -u origin master
# We can create a bundle of files with gatsby
# which collects all files necessary to deploy our website
# The created bundle/folder is called public/
# it will be our artifact
gatsby build
# We now open the local static-website project
# with ane editor, say VS Code,
# and create and edit a pipeline file:
# .gitlab-ci.yml
```

### 2.1 First Gitlab Pipeline for the Static Website

Now, we'd like to pack all this into a itlab pipeline.
Notes for the CI/CD pipeline on Gitlab:
- The file `static-website/package.json` defined all the dependencies needed for our website, which are installed in the git-ignored folder `static-website/node_modules`.
- Since we use docker containers for CI, we actively need to install the dependencies every time, thus, the installation commands must appear in our `.gitlab-ci.yml` file.
- Additionally, we need to take a `node` docker image for our pipeline, otherwise a default `ruby` image is used (which has no `node` platform).
- If we want the static website bundle, we need to save it as an artifact.


`.gitlab-ci.yml`:
```yaml
build the website:
  image: node
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby build
  artifacts:
    paths:
      - ./public
```

The pipeline should be executed automatically.
The execution can be check on the vertical menu `Pipeline`, and the artifacts that are generated can be downloaded/browsed.


### 2.2 Adding a Test Stage

A job that is successful returns `0`, otherwise a value in the range `1-255`.
We can use this feature for executing unit tests.

In our simple example, we check whether `public/index.html` contains a certain string: `Gatsby`:

```bash
cd public
grep "Gatsby" index.html
# Grep in quiet mode: no output
grep "Gatsby" index.html
# To get the return status of the last command
echo $? # 0
```

Note that:
- The default image is `ruby`. We need to specify the image if we want another one that `ruby`; often the small/lightweight Linux distro `alpine` is chosen.
- Gatsby can serve the main website on `localhost:9000`, so we can check that using `curl` and `grep`; note that we need to add `tac` in-between so that `grep` receives the complete website.
- If a command is expected to block the terminal because it runs forever (eg., `gatsby serve`), we should add `&` to it. That way, the command is run in the background. However, we might need to add a `sleep` before the next command in case it depends on the previous blocking one.

`.gitlab-ci.yml`:
```yaml
stages:
  - build
  - test

build the website:
  image: node
  stage: build
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby build
  artifacts:
    paths:
      - ./public

test artifact:
  image: alpine
  stage: test
  script:
    - grep -q "Gatsby" ./public/index.html

test website:
  image: node
  stage: test
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby serve &
    - sleep 3
    - curl "http://localhost:9000" | tac | tac | grep -q "Gatsby"  
```

### 2.3 Deploying Our Website

The service offered by [surge.sh](https://surge.sh/) is used to deploy the static website. Surge is a cloud platform for serverless deployments; serverless means basically: we don't care how it works, we just execute simple commands for deployment and the server takes care of everything. Surge works very nicely with statc websites.

```bash
# install surge
npm install --global surge
# go to the folder where the index.html is
cd ~/git_repositories/static-website/public
surge
# we login/create a surge account
# email
# pw
# a random name for the static website is suggested
# jumbled-can.surge.sh
# later on, we can change the name to a name acquired on namecheap.com
# or another service!
# firefox(jumbled-can.surge.sh)
```

### 2.4 Secrets

Secrets can be used to manage credentials; we don't want to commit them to git repositories. For that, environment variables are generated in the Gitlab web interface which are then available on the docker containers that perform the CI/CD pipeline. Note that many CLI tools rely on environment variables that need to be set on the Gitlab web interface.

```bash
# go to the folder where the index.html is and surge was run
cd ~/git_repositories/static-website/public
# generate a token
surge token
# On `gitlab.com/...`, vertical menu:
# Settings, CI/CD, Variables. We add the following:
# SURGE_LOGIN <our email> 
# SURGE_TOKEN <our newly generated token>
```

Now, we can create a new stage in which we deploy our website!
Note the following remarks:
- Since most jobs use the image `node`, we can put it up front and define `image` in jobs where it should be different than `node`.
- The environment variables for logging into surge are already set on the Gitlab web interface.
- We need to enconde the name of our website; either we use the name given by surge at the biginning or one we have specified on our surge account.
- The job(s) of the `deploy` stage are executed only after the previous have been executed correctly.

`.gitlab-ci.yml`:
```yaml
image: node

stages:
  - build
  - test
  - deploy

build the website:
  stage: build
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby build
  artifacts:
    paths:
      - ./public

test artifact:
  image: alpine
  stage: test
  script:
    - grep -q "Gatsby" ./public/index.html

test website:
  stage: test
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby serve &
    - sleep 3
    - curl "http://localhost:9000" | tac | tac | grep -q "Gatsby"  

deploy to surge:
  stage: deploy
  script:
    - npm install --global surge
    - surge --project ./public --domain jumbled-can.surge.sh
```

## Section 3: Gitlab CI Fundamentals


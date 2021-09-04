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
`Node.js` is a Javascript runtime environment able to run Javascript without needing a browser; `npm` is the Node Package Manager.

Installation:
```bash
brew install node
node -v # we should see the version
npm -v # we should see the version
```


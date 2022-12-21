# CI/CI Pipelines with Github Actions

This guide was prepared after following these courses:

- The Udemy course by Ali Alaa, which deals with Github Actions: [The Complete GitHub Actions & Workflows Guide](https://www.udemy.com/course/github-actions/).
- The Udacity Nanodegree, which deals with Github Actions [Machine Learning DevOps Engineer](https://www.udacity.com/course/machine-learning-dev-ops-engineer-nanodegree--nd0821).


Mikel Sagardia, 2022. 
No warranties.

## Table of Contents

- [CI/CI Pipelines with Github Actions](#cici-pipelines-with-github-actions)
  - [Table of Contents](#table-of-contents)
  - [1. Introduction](#1-introduction)
    - [Example 1: Template for Python App Integration](#example-1-template-for-python-app-integration)
    - [Example 2:](#example-2)
    - [Example 3:](#example-3)

## 1. Introduction

Some basic definitions:

- CI = Continuous Integration = Automated Testing. In other words, we make sure that any change we implement can be integrated in the code base without breaking it, i.e., the new implementations are integrable. CI makes possible to deploy our code any time, i.e., continuous deployment!
- CD = Continuous Deployment = Deploy code/applications verified by CI automatically, without time gaps from the implementation integration. That way, the latest version of an app is always available to the users.

Github actions can do both CI and CD. We choose a repository and click on `Actions` menu. We have some pre-built workflows:

- Deployment: automated deployment to AWS EC2
- Integration: automated linting (`flake8`), testing (`pytest`)
- Greeting users when users do Pull Requests.
- etc.

Interesting links:

- [GitHub's overview on Actions](https://github.com/features/actions)
- [GitHub's docs on Actions](https://docs.github.com/en/actions)
- [A repository of GitHub's starter workflows](https://github.com/actions/starter-workflows)

### Example 1: Template for Python App Integration

The following is a pre-built template for CI of a python package. If any of the steps fail we don't reach the end point and the automation has failed.

```yaml
name: Python package # Name of the Action.

on: [push] # When this action runs.

jobs: # tasks that occur in the automation
  build:

    runs-on: ubuntu-latest # Which OS this runs on, you can also build on Windows or MacOS.
    strategy:
      matrix: # several builds
        python-version: [3.6, 3.7, 3.8] # You can build against multiple Python versions.

    steps: # steps that occur in the job
    - uses: actions/checkout@v2 # Calling a pre-built GitHub Action which allows your Action to access your repository.
    - name: Set up Python ${{ matrix.python-version }} # Name of an action that sets up Python.
      uses: actions/setup-python@v2 # A pre-built GitHub Action that sets up a Python environment.
      with:
        python-version: ${{ matrix.python-version }}
    - name: Install dependencies # The first step that isn't just calling another action.
      run: | # bash commands, multiline: |
        python -m pip install --upgrade pip # Upgrade pip to the latest version.
        pip install pytest # Install pytest.
        if [ -f requirements.txt ]; then pip install -r requirements.txt; fi # If we have a requirements.txt, then install it.
    - name: Test with pytest # Final action which runs pytest. If any test fails, then this Action fails.
      run: | # bash commands, multi-line
        pytest
```

### Example 2: 


### Example 3:

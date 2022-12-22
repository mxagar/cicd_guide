# CI/CI Pipelines with Github Actions

This guide was prepared after following these courses:

- The Udemy course by Ali Alaa, which deals with Github Actions: [The Complete GitHub Actions & Workflows Guide](https://www.udemy.com/course/github-actions/).
- The Udacity Nanodegree, which deals with Github Actions [Machine Learning DevOps Engineer](https://www.udacity.com/course/machine-learning-dev-ops-engineer-nanodegree--nd0821).

This is a very practical guide; if you're looking for a more detailed guide that builds from the ground up, you can check the co-located guide on Gitlab Pipelines: [`gitlab_ci_cd_howto.md`](gitlab_ci_cd_howto.md).

Mikel Sagardia, 2022. 
No warranties.

## Table of Contents

- [CI/CI Pipelines with Github Actions](#cici-pipelines-with-github-actions)
  - [Table of Contents](#table-of-contents)
  - [1. Introduction](#1-introduction)
    - [Example 1: Template for Python App Integration](#example-1-template-for-python-app-integration)
    - [Example 2: Test Actions with Templates for Simple Automations or Python Apps](#example-2-test-actions-with-templates-for-simple-automations-or-python-apps)

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

The following is a pre-built template for CI of a python package. If any of the steps fail we don't reach the end point and the automation has failed. See Example 2 to understand how such a YAML file is generated automatically.

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

### Example 2: Test Actions with Templates for Simple Automations or Python Apps

I created a repository: [test_actions](https://github.com/mxagar/test_actions).

Then:

- `Actions` menu.
- Select **Simple Workflow**, Configure.
- A default workflow YAML is generated: `.github/workflows/blank.yaml`
  - This workflow is similar to the one in the example 1.
  - However, this workflow is triggered on pushes and pull requests.
  - The workflow job checks out our repo and performs some echos (single-line and multi-line)
- Start commit: add a nice commit message.
- Now, the workflow is created and active.
- If we go to `Actions`, we see it's running.

Now, we create another workflow: **Stale**

- `Actions` menu, **New Workflow**.
- Select **Stale**, Configure.
- A workflow YAML is generated: `.github/workflows/stale.yaml`
    - This workflow warns and then closes issues and PRs that have had no activity for a specified amount of time.
    - This workflow runs on a schedule not an action/event
    - The schedule is defined with the syntax of a cron job.
    - The necessary arguments can be checked in the template action/workflow repo:
      - [https://github.com/actions/stale](https://github.com/actions/stale)
    - Always check the template repos!

Other workflows: **Python App**

- The process steps are the same.
- I tested this workflow: **Python application: Create and test a Python application.**
  - A workflow YAML is generated: `.github/workflows/python-app.yaml`
  - The workflow performs these steps in the main job:
    - Install dependencies: `requirements.txt`
    - Lint with `flake8`
    - Test with `pytest`
- Obviously, the necessary files need to be created:
  - `requirements.txt`
  - `app.py`
  - `test_app.py`
- We can see in the Actions tab all the run/executed workflows and their logs!

File contents


`python-app.yaml`, automatically generated:

```yaml
# This workflow will install Python dependencies, run tests and lint with a single version of Python
# For more information see: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-python

name: Python application

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

permissions:
  contents: read

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - name: Set up Python 3.10
      uses: actions/setup-python@v3
      with:
        python-version: "3.10"
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install flake8 pytest
        if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
    - name: Lint with flake8
      run: |
        # stop the build if there are Python syntax errors or undefined names
        flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
        # exit-zero treats all errors as warnings. The GitHub editor is 127 chars wide
        flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
    - name: Test with pytest
      run: |
        pytest

```

`requirements.txt`:

```
numpy
```

`app.py` and `test_app.py`:

```python
### app.py
import numpy as np

def square(n):
	return np.power(n,2)


### test_app.py
import pytest
from app import square

@pytest.fixture
def input_value():
	return 2

def test_square_gives_correct_value(input_value):
    subject = square(input_value)
    assert subject == 4

```


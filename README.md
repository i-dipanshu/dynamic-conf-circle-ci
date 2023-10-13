# Advanced Dynamic Configuration - Circle CI

## Path Filtering 
```yaml
version: 2.1

# This allows you to use CircleCI's dynamic configuration feature
setup: true

# The path-filtering orb is required to continue a pipeline based on
# the path of an updated file_set
orbs:
  path-filtering: circleci/path-filtering@1.0.0

workflows:
  # The always-run workflow is always triggered, regardless of the pipeline parameters.
  always-run:
    jobs:
      # The path-filtering/filter job determines which pipeline
      # parameters to update.
      - path-filtering/filter:
          # I’ve defined my base-revision to be the main branch.
          # Any commit pushed to my development branch will
          # inherently yield a diff when compared to my main branch.
          # Once I’m ready to deploy changes to production,
          # development is merged into main and pushed to GitHub.
          # The orb is smart enough to detect commits pushed to the same branch as the base-revision.
          # In this scenario, the orb compares the new commit against the previous commit on the main branch (aka HEAD~1).
          base-revision: main

          # 3-column, whitespace-delimited mapping. One mapping per line:
          # <regex path-to-test> <parameter-to-set> <value-of-pipeline-parameter>
          mapping: |
            backend/.* run-backend-workflow true
            frontend/.* run-frontend-workflow true

          # This is the path of the configuration that will trigger once
          # path filtering and pipeline parameter value updates are complete.
          config-path: .circleci/workflow.yml
```

## 2. Sample Workflow for monolithic repo
```yaml
version: 2.1

parameters:
  run-backend-workflow:
    type: boolean
    default: false
  run-frontend-workflow:
    type: boolean 
    default: false

executors:
  ubuntu_machine:
    machine:
      image: ubuntu-2204:2023.07.2

jobs:

  build_frontend:
    executor: ubuntu_machine
    steps:
      - checkout
      - run:
          name: Builds the frontend
          command: |
            echo "Frontend Build triggered"
            cat ./frontend/sample.txt

  deploy_frontend:
    executor: ubuntu_machine
    steps:
      - checkout
      - run:
          name: Builds the frontend
          command: echo "Frontend Deployment triggered"

  build_backend:
    executor: ubuntu_machine
    steps:
      - checkout
      - run:
          name: Builds the backend
          command: echo "Backend Build triggered"
          
  deploy_backend:
    executor: ubuntu_machine
    steps:
      - checkout
      - run:
          name: Builds the backend
          command: echo "Backend Deploy triggered"

workflows:
  frontend_workflow:
    when:
      <<pipeline.parameters.run-frontend-workflow>>
    jobs:
      - build_frontend
      - deploy_frontend:
          requires:
            - build_frontend
          filters:
            branches:
              only: main
  
  backend_workflow:
    when:
      <<pipeline.parameters.run-backend-workflow>>
    jobs:
      - build_backend
      - deploy_backend:
          requires:
            - build_backend
          filters:
            branches:
              only: main
```

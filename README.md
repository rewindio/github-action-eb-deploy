# github-action-eb-deploy

This repository has reusable workflows to drop the required boilerplate for deploying Elastic Beanstalk in multiple reposotories.

## Overall Usage

In general, you will preprocess the environment, create the deployment zip, and then deploy the app. The steps here are laid out to assist you to populate your yaml properly to avoid the boilerplate that comes along with doing this in many repos.

### preprocess-eb-environment Usage

This is a Github reusable workflow to set up the Elastic Beanstalk environment by cloning and copying in the extensions & hooks requested by the calling project.

### create-zip Usage

This is a GitHub reusable workflow that will package up a basic deployment zip for you

### preprocess-eb-environment.yml Usage

This is a GitHub reusable workflow that will deploy the application for you, given the required inputs, and integrate with Slack on success or failure for notifications.

#### Required Inputs

| Key | Value | Secret | Required |
| ------------- | ------------- | ------------- | ------------- |
| application_name | The name of the Elastic Beanstalk application to deploy to | No | Yes |
| aws_region | The region to deploy to (default: us-east-1) | No | No |
| deployment_environment_name | The deployment environment, either staging or production, for printing purposes only. | No | Yes |
| environment_name_list | A JSON list of Elastic Beanstalk environment name. Must have escaped quotes on each list element. | No | Yes |
| version_label | The Elastic Beanstalk version label | No | Yes |
| EB_AWS_ACCESS_KEY_ID | The AWS Access Key. [More info here.](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html) | Yes | Yes |
| EB_AWS_SECRET_ACCESS_KEY_ID | The AWS Secret Access Key. [More info here.](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html) | Yes | Yes |

## `workflow.yml` Example

Place in a `.yml` file such as this one in your `.github/workflows` folder. [Refer to the documentation on workflow YAML syntax here.](https://help.github.com/en/articles/workflow-syntax-for-github-actions)

```yaml
name: Deploy
on: push

jobs:
  preprocess-eb-environment:
    name: "Preprocess EB environment"
    uses: rewindio/github-action-eb-deploy/preprocess-eb-environment.yml
    secrets:
      # Please ensure the right-hand side is appropriately named for your repo &/ env
      BUNDLE_RUBYGEMS__PKG__GITHUB__COM: ${{ secrets.BUNDLE_RUBYGEMS__PKG__GITHUB__COM }}
      BUNDLE_GEMS__CONTRIBSYS__COM: ${{ secrets.CONTRIBSYS_TOKEN }}
      CONTAINER_REGISTRY_PAT: ${{ secrets.CONTAINER_REGISTRY_PAT }}
  
  create-deployment-zip:
    name: "Create Deployment Zip"
    uses: rewindio/github-action-eb-deploy/create-zip.yml
    needs: [ preprocess-eb-environment ]

  deploy-staging:
    name: "Deploy Staging environments"
    uses: rewindio/github-action-eb-deploy/deploy-eb.yml
    needs: [ create-deployment-zip ]
    with:
      deployment_environment_name: "staging"
      application_name: "My App Name"
      aws_region: "us-east-2"
      # This must parse later as JSON, so we need to add escaped quotes on each element
      environment_name_matrix: "[ \"my-env-1\", \"my-env-2\" ]"
      # Please note that only `github` and `needs` variables are accessible here 
      # c.f.: https://docs.github.com/en/actions/learn-github-actions/contexts#context-availability
      version_label: "my-app-${{ github.sha }}"
```

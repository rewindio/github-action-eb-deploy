# github-action-eb-deploy

This repository has reusable workflows to drop the required boilerplate for deploying Elastic Beanstalk in multiple reposotories.

## Overall Usage

In general, you will preprocess the environment, create & deploy the deployment zip, and then deploy the app. The yml files noted in the headings below do this for you in this order. The steps below those are laid out to assist you to populate your yaml properly to avoid the boilerplate that comes along with doing this in many repos.

## preprocess-eb-environment.yml Usage

This is a Github reusable workflow to set up the Elastic Beanstalk environment by cloning and copying in the extensions & hooks requested by the calling project.

## pack-and-deploy-appversion.yml Usage

This is a GitHub reusable workflow that will package up a basic single-gem zip for you and deploy the application version to Elastic Beanstalk.

## deploy-env.yml Usage

This is a GitHub reusable workflow that will deploy a given application version to a specific environment for you. It integrates with Slack to send you notifications on success or failure.

### Input Descriptions

| Key | Value |
| ------------- | ------------- |
| application_name | The name of the Elastic Beanstalk application to deploy to |
| aws_region | The region to deploy to (default: us-east-1) |
| docker_ruby_version | The version of Ruby to be used by the job container. |
| environment_name_list | A JSON list of Elastic Beanstalk environment names to deploy the application into. They must all be in one region. |
| version_label | The Elastic Beanstalk version label |

## Secret Description

| Key | Value |
| ------------- | ------------- |
| BUNDLE_RUBYGEMS__PKG__GITHUB__COM |  |
| BUNDLE_GEMS__CONTRIBSYS__COM | |
| EB_AWS_ACCESS_KEY_ID | [More info here.](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html) | Yes | Yes |
| EB_AWS_SECRET_ACCESS_KEY | The AWS Secret Access Key. [More info here.](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html) |

## `workflow.yml` Example

Place in a `.yml` file such as this one in your `.github/workflows` folder. [Refer to the documentation on workflow YAML syntax here.](https://help.github.com/en/articles/workflow-syntax-for-github-actions)

```yaml
name: Deploy
on: push

jobs:
  preprocess-eb-environment:
    name: "Preprocess EB environment"
    uses: rewindio/github-action-eb-deploy/preprocess-environment.yml@v1.0.1
    # You may specify the docker ruby version here
    # with:
    #   docker_ruby_version: "2.6.8" # Optional, this is the default
    secrets:
      # Please ensure the right-hand side is appropriately named for your repo &/ env
      BUNDLE_RUBYGEMS__PKG__GITHUB__COM: ${{ secrets.BUNDLE_RUBYGEMS__PKG__GITHUB__COM }}
      BUNDLE_GEMS__CONTRIBSYS__COM: ${{ secrets.CONTRIBSYS_TOKEN }}
      CONTAINER_REGISTRY_PAT: ${{ secrets.CONTAINER_REGISTRY_PAT }}

   package-and-deploy-eb-app-version:
    name: "Package & Deploy EB App Version"
    uses: rewindio/github-action-eb-deploy/.github/workflows/pack-and-deploy-appversion.yml@v1.0.1
    needs: [ preprocess-eb-environment ]
    with:
      application_name: "My App Name"
      # docker_ruby_version: 2.6.8 # Optional; this is the default
      version_label: "my-version-label"
    secrets:
      EB_AWS_ACCESS_KEY_ID: ${{ secrets.STAGING_AWS_ACCESS_KEY_ID }}
      EB_AWS_SECRET_ACCESS_KEY: ${{ secrets.STAGING_AWS_SECRET_ACCESS_KEY }}

  deploy-staging:
    name: "Deploy Staging environments"
    uses: rewindio/github-action-eb-deploy/deploy-eb.yml@v1.0.1
    needs: [ package-and-deploy-eb-app-version ]
    with:
      # Must match above, and please note that only `github` and `needs` variables are accessible here
      # c.f.: https://docs.github.com/en/actions/learn-github-actions/contexts#context-availability
      application_name: "My App Name"
      # Must match above
      version_label: "my-version-label"
      aws_region: "us-east-2"
      # This must parse later as JSON, so we need to use single quotes or add escape all nested double quotes
      environment_name_matrix: '[ "my-env-1", "my-env-2" ]'
    secrets:
      EB_AWS_ACCESS_KEY_ID: ${{ secrets.STAGING_AWS_ACCESS_KEY_ID }}
      EB_AWS_SECRET_ACCESS_KEY: ${{ secrets.STAGING_AWS_SECRET_ACCESS_KEY }}
      # Secrets are not accessible unless they are shared, so we need these three even though they are redundant
      DEPLOY_FAILURES_SLACK_WEBHOOK_URL: ${{ secrets.DEPLOY_FAILURES_SLACK_WEBHOOK_URL }}
      DEPLOY_SUCCESS_SLACK_WEBHOOK_URL: ${{ secrets.DEPLOY_SUCCESS_SLACK_WEBHOOK_URL }}
      LOOKUP_USER_EMAIL_SLACK_TOKEN: ${{ secrets.LOOKUP_USER_EMAIL_SLACK_TOKEN }}

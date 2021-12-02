# github-action-eb-deploy

This repository has reusable workflows to drop the required boilerplate for deploying Elastic Beanstalk in multiple reposotories.

## Overall Usage

In general, you will compile the artifact, optionally upload the deployment zip within a region as an app version, and then deploy the app version to environments. The yml files noted in the headings below do this for you in this order. The steps below those are laid out to assist you to populate your yaml properly to avoid the boilerplate that comes along with doing this in many repos.

## compile-artifact.yml Usage

This is a Github reusable workflow to set up the Elastic Beanstalk environment by cloning and copying in the extensions & hooks requested by the calling project, and then package up a basic single-gem zip, uploaded to GitHub as "eb-app-zip" with a filename of "deploy.zip".

## deploy-env.yml Usage

This is a GitHub reusable workflow that will deploy a given application version to a set of environments for you. It integrates with Slack to send you notifications on success or failure.

It is also used for pre-loading app versions within a region. This is done by reducing the "env" column from your intended deploy_matrix and keeping one copy of any row which appears twice or more. The hard example below will make this computation far clearer.

### Inputs & Secrets

Each yml requires specific inputs and secrets. Please read the inputs and secrets list in the yml you are looking to use for descriptions and whether or not they are required for execution.

## `workflow.yml` Example

Place in a `.yml` file such as this one in your `.github/workflows` folder. [Refer to the documentation on workflow YAML syntax here.](https://help.github.com/en/articles/workflow-syntax-for-github-actions)

```yaml
name: Deployment
on: push

jobs:
  compile-artifact:
    name: "Compile artifact zip"

    # Please replace all X.Y.Z values with the latest tag.
    uses: rewindio/github-action-eb-deploy/.github/workflows/compile-artifact.yml@vX.Y.Z
    # You may specify the docker ruby version here.
    # with:
    #   docker_ruby_version: "2.6.8" # Optional, this is the default
    secrets:
      # Please ensure the right-hand side is appropriately named for your repo &/ env.
      BUNDLE_RUBYGEMS__PKG__GITHUB__COM: ${{ secrets.BUNDLE_RUBYGEMS__PKG__GITHUB__COM }}
      BUNDLE_GEMS__CONTRIBSYS__COM: ${{ secrets.CONTRIBSYS_TOKEN }}
      CONTAINER_REGISTRY_PAT: ${{ secrets.CONTAINER_REGISTRY_PAT }}

   # SKIP THIS IF: your deploy matrix below does not have rows which only differ in the "env" column.
   upload-app-version-staging:
    name: "Upload app version(s) to staging"
    needs: [ compile-artifact ]

    # NOTE: this yml is the same workflow as the deploy below, it is simply the "deploy_matrix" usage that changes.
    uses: rewindio/github-action-eb-deploy/deploy-env.yml@vX.Y.Z
    with:
      # This matrix must be a valid JSON object list. Use single quotes around a single line matrix.
      # Double quoting a single line matrix only works by escaping all inner quotes with a backslash (\).
      # The objects must have 'app' name and 'region'. For more complex matrices, see the comments below.
      #
      # We submit the MyApp within us-east-1 here because it is used twice (or more) below in the final deploy_matrix.
      deploy_matrix: '[ { "app" : "MyApp", region: "us-east-1" } ]'
      version_label: "my-version-label"
    secrets:
      EB_AWS_ACCESS_KEY_ID: ${{ secrets.STAGING_AWS_ACCESS_KEY_ID }}
      EB_AWS_SECRET_ACCESS_KEY: ${{ secrets.STAGING_AWS_SECRET_ACCESS_KEY }}
      # Secrets are not accessible unless they are shared, so we need these three even though they are redundant.
      DEPLOY_FAILURES_SLACK_WEBHOOK_URL: ${{ secrets.DEPLOY_FAILURES_SLACK_WEBHOOK_URL }}
      DEPLOY_SUCCESS_SLACK_WEBHOOK_URL: ${{ secrets.DEPLOY_SUCCESS_SLACK_WEBHOOK_URL }}
      LOOKUP_USER_EMAIL_SLACK_TOKEN: ${{ secrets.LOOKUP_USER_EMAIL_SLACK_TOKEN }}

  deploy-staging:
    name: "Deploy staging environments"
    needs: [ upload-app-version-staging ] # or depend directly on compile-artifact, if skipping this

    uses: rewindio/github-action-eb-deploy/deploy-env.yml@vX.Y.Z
    with:
      # Please note the following gotchas to working with this matrix
      # 1. It must parse as a valid JSON object list. Single-line matrices can use single-quotes (see above).
      # 2. Each object requires all three parameters, as of the current version.
      # 3. In order to avoid a race condition and inefficiencies, any app name that appears twice must exist
      #      in the matrix of uploaded app versions above. Otherwise the second upload will crash the workflow.
      # 4. Most variable contexts are not usable here -- only `github` and `needs` variables can be resolved.
      #      c.f.: https://docs.github.com/en/actions/learn-github-actions/contexts#context-availability
      deploy_matrix: |
        [
          { "app": "MyApp", "env": "my-env-1", region: "us-east-1" },
          { "app": "MyApp", "env": "my-env-2", region: "us-east-1" },
          { "app": "MyApp", "env": "my-env-3", region: "ca-central-1" },
          { "app": "YourApp", "env": "my-env-4", region: "eu-west-2" },
          { "app": "TheirApp", "env": "my-env-5", region: "eu-east-1" },
        ]
      # Note in the above matrix that the first two rows only differ in the "env" column. This is why we need the step above.
      version_label: "my-version-label" # Ignoring edge-cases, this must match the above
    secrets:
      EB_AWS_ACCESS_KEY_ID: ${{ secrets.STAGING_AWS_ACCESS_KEY_ID }}
      EB_AWS_SECRET_ACCESS_KEY: ${{ secrets.STAGING_AWS_SECRET_ACCESS_KEY }}
      DEPLOY_FAILURES_SLACK_WEBHOOK_URL: ${{ secrets.DEPLOY_FAILURES_SLACK_WEBHOOK_URL }}
      DEPLOY_SUCCESS_SLACK_WEBHOOK_URL: ${{ secrets.DEPLOY_SUCCESS_SLACK_WEBHOOK_URL }}
      LOOKUP_USER_EMAIL_SLACK_TOKEN: ${{ secrets.LOOKUP_USER_EMAIL_SLACK_TOKEN }}

  # In order to run for production, please repeat both of the final two job blocks above (upload & deploy).
  #
  # Under each prod job heading, consider also adding:
  #  if: github.ref == 'refs/heads/main' && github.event_name == 'push' # Only run on pushes to main

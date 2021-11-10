# github-action-setup-eb-env

This is a Github action to set up the Elastic Beanstalk environment by cloning and copying in the extensions requested by the calling project

## Usage

### `workflow.yml` Example

Place in a `.yml` file such as this one in your `.github/workflows` folder. [Refer to the documentation on workflow YAML syntax here.](https://help.github.com/en/articles/workflow-syntax-for-github-actions)

```yaml
name: Deploy
on: push

jobs:
  call-workflow-1:
    uses: rewindio/github-action-eb-deploy/setup-eb-ext.yaml
```

### Environment Variables

The following tables describe the environment variables that are used.

#### Configurable Variables 

| Key | Value | Type | Required |
| ------------- | ------------- | ------------- | ------------- |
| `AWS_REGION` | The region where you created your bucket in. For example, `us-east-1`. Defaults to `us-east-1` if not specified | `env` | **No** |

#### Required Secret Variables

The following variables should be added as "secrets" in the action's configuration.

| Key | Value | Type | Required |
| ------------- | ------------- | ------------- | ------------- |
| `AWS_ACCESS_KEY_ID` | Your AWS Access Key. [More info here.](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html) | `secret` | **Yes** |
| `AWS_SECRET_ACCESS_KEY` | Your AWS Secret Access Key. [More info here.](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html) | `secret` | **Yes** |

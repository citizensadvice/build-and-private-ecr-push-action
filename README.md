# Build and private ECR push action

Build and push a Docker image to a private ECR repository in one action. This action can only be used on **private** Github repositories. For public repositories, please use [this action.](https://github.com/citizensadvice/build-and-push-action)

By default it will push the image with three different tags:

- **Commit sha**, prepended with `dev_` e.g. `dev_e1b18d15d2b5558d275920ab5b5650a5c36bb1ee`
- **Branch** or **Tag** on push events or **PR Source Branch** for pull request events, e.g. `main`, `v1.0.0`, `feature`

When marked as a production image using `prod_image: true`

- **Commit sha**, e.g. `e1b18d15d2b5558d275920ab5b5650a5c36bb1ee`
- **Branch** or **Tag** on push events or **PR Source Branch** for pull request events, e.g. `main`, `v1.0.0`, `feature`
- **latest**


Please note that the ECR repository must be created beforehand. Repository creation is not currently within the scope of this Action.

## Cache

This action will automatically cache Docker image layers to the Github cache, however this feature is experimental.

## Authentication

In order to give a repository the AWS permissions required to run this action, the Github repository name and the ECR repository ARN **must** be added to the `PrivateECRPush` configuration in the [AWS OIDC CDK](https://github.com/citizensadvice/aws-oidc-cdk) repository and then deployed. Please send a message to `#devops-support` in Slack for help with this.

## Inputs

| Input                | Description                                                                                                                                                         | Required | Default                                         |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ----------------------------------------------- |
| `role_arn`           | A role to assume with permission to push to the Private ECR repository                                                                                              | No       | 'arn:aws:iam::979633842206:role/PrivateECRPush' |
| `dockerfile_context` | The path to the Dockerfile context. Will default to the root of the project                                                                                         | No       | `.`                                             |
| `repository_name`    | The name of the image repository. Is used to name the docker image. Must may contain lowercase and uppercase letters, digits, underscores, periods and dashes only. | Yes      |                                                 |
| `auth_token`         | A token with permission to clone the repository. Will usually be GITHUB_TOKEN                                                                                       | Yes      |                                                 |
| `multiarch_build`    | Allow for multi-arch builds. When `'disabled'` only builds `linux/amd64` images, when `'enabled'` also builds `linux/arm64` images                                  | No       | `'enabled'`                                     |
| `push_after_build`   | Push the image after building it. Useful if you need to run tests on the image before you push it                                                                   | No       | `true`                                          |
| `dockerfile`         | The name of the dockerfile to be built.         | No       | `Dockerfile`                                    |
| `build-args`         | List of build-time variables.   | No       |                                    |
| `prod_image`         | Mark the image as a production image. Adds `latest` tag. When false, adds `dev_` to hash tag.                                                                       | No       | false                                           |

The user associated with the `aws_access_key` must have permission to push, update and read the private repository in question.

Due to the fact that the action will be running in an `amd64` environment (Also known as `x86_64`), building the image with multi-arch enabled will cause the build times to increase **significantly**, so use only when required.

## Outputs

| Name           | Description                                                                   |
| -------------- | ----------------------------------------------------------------------------- |
| `image_id`     | The ID of the built image                                                     |
| `image_digest` | The digest of the built image                                                 |
| `image_tags`   | A CSV list of the image tags, in the order `commit sha`,`branch/tag`,`latest` |

## Example usage

Located in `.github/workflows/build-and-push.yml`:

```yaml
name: Build and private ECR push action
on:
  push:
    branches:
      - main
      - "[0-9]*"
      - "v[0-9]*"
    tags:
      - v*

jobs:
  build:
    name: Publish image
    runs-on: ubuntu-22.04
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Build and push to ECR
        uses: citizensadvice/build-and-private-ecr-push-action@v1
        with:
            dockerfile_context: '.'
            repository_name: <REPOSITORY NAME HERE>
            auth_token: ${{ secrets.GITHUB_TOKEN }}
            prod_image: true
```

The `on` conditions and branch matching can be changes to whatever suites your team best. If you require assistance implementing a custom solution, please send a message to `#devops-support` in Slack.

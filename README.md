# Raygun Github Actions

Deploy workflow examples for Craft CMS atomic deploy to VPS, webhook API builds, Craft CMS plugin releases, etc.

**Note:** do not link directly to workflows in this repository. This repo is being made public to provide examples of a typical VPS atomic deployment for Craft CMS using Github Actions, as well as other deployments used at [Raygun](https://madebyraygun.com). If you link to these workflows directly from your repo, your deployment may break at any time.

This assumes some basic knowledge of Github [environments](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment) and [secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions). 

In our case, the SSH key is set as a secret [at the organization level](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-an-organization), but this can also be managed at the repo or environment level. All other variables are managed at the environment level.

## Craft CMS

This workflow is comprised of two parts: an environment/branch based workflow which sets your secrets and environment variables, and a reusable deploy workflow. Setting your workflow up this way allows you to set unique environment variables for each deploy target and reuse the same steps for each deploy.

1. Copy `.github/workflows/craft/deploy.yaml` and `.github/workflows/craft/production-example.yaml` to the `.github/workflows` directory in your repository. Rename `production-example.yaml` to the name of the environment where it will be used, e.g. `production.yaml`, `staging.yaml`, etc. You can create as many of these environment workflows as you need, each can be linked to a particular branch (or branches) and environment in your repo.

1. Update the `branches` param of your environment workflow to [determine which branches](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#using-filters) should trigger this workflow on push.

    ```yml
    on:
      push:
        branches:
          - 'main'
          - 'releases/**'
          ...
    ```

1. Update the `uses` keyword to reference your local repo's version of the `deploy.yaml` script. This will probably look like 
    ```yml
    jobs:
      call-workflow-passing-data:
        uses: ./.github/workflows/deploy.yaml
    ```
    You can also [reference a reusable workflow from a separate repository](https://docs.github.com/en/actions/using-workflows/reusing-workflows#calling-a-reusable-workflow) and use it in multiple projects.

1. Update the `environment` param of your environment workflow to point to a specific [environment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment) in your repo where your secrets and variables are held.

1. Create the environment in your repo (if it doesn't already exist) and make sure it has the following variables defined:

    * `NODE_VERSION`: e.g. `15` (only useful if you have an NPM build step)
    * `PHP_VERSION`: e.g. `81` (only useful if you want to restart a related PHP service *and* your ssh account has sudoers rights to that service. This is service dependent and will probably need to be modified in `deploy.yaml`.)
    * `REMOTE_PROJECT_ROOT`: the file path to your project root, e.g. `/home/raygun/webapps/project-name`
    * `SSH_HOST`: e.g. `staging.example.com`
    * `SSH_USER`
    * `PERSISTENT_DIRS`: a comma separated list of directories in the root project folder that will be symlined in the public directory. Useful for local asset folders that should persist between deployments.

1. Review the steps in `deploy.yaml`, some may not be relevant for your project or environment. Edit as necessary.

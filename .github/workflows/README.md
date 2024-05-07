# About our workflows

This repository uses different workflows to automatically updates the published websites

- <https://quarto.org/> - The documentation for Quarto, published from the `main` branch
- <https://prerelease.quarto.org/> - The documentation for Quarto prerelease, published from the `prerelease` branch

It also configures some website preview for pull requests, which are published to Netlify as deploy previews. The use non-production deploy and can be consulted to `deploy-preview-<PR number>.quarto.org`. 

Note that technically, <prerelease.quarto.org> is also a deploy preview on Netlify (non-production website) but with a fixed URL.

## Workflow files: What they do

- `publish.yml` - The workflow that publishes the documentation from `main` and `prerelease` branches to netlify. 
  - For stable release version, the doc is deployed from `main`, and published using `quarto publish netlify` command to production website on Netlify.
  - For prerelease version, the doc is deployed from `prerelease`, and published using Github Action `nwtgck/actions-netlify` which use Netlify API to deploy to a non-production website on Netlify.

- `preview.yml` - The workflow that creates a deploy preview for pull requests. It uses the same action `nwtgck/actions-netlify` to deploy to a non-production website on Netlify. The URL is `deploy-preview-<PR number>.quarto.org`.
  - PR previews are configured for any PR to `main` or`prerelease` branches.
  - They are automically created and updated when the PR is created and updated by a user with Contributor role. 
  - For external PR, the preview can be trigger by adding a comment `/deploy-preview` on the PR.

- `update-downloads.yml` - This workflow is triggered by a cron schedule. It retrieves information about latest release and prerelease on `quarto-dev/quarto-cli` repository and updates the download links on the website.
  - If there is a new version detected, it will commit the modified files and trigger a deploy of the website calling `publish.yml` workflow with `workflow_call` event trigger.
  - This applies to both `main` and `prerelease` branches
    - For `main` branch, we use `stefanzweifel/git-auto-commit-action` to detect changes and commit them to the `main` branch
    - For `prerelease` branch, we use `git cherry-pick` the commit from main following previous above step. 
    - Then we trigger the `publish.yml` workflow with `workflow_call` event trigger for each of the branch.

- `upload-index.yml` - This workflow is triggered by a cron schedule. It updates the index for Algolia search engine, which powers the site search. 

## Netlify Configurations

- This repo has a `_redirects` file in the root directory. Otherwise, configuration are made in NETLIFY UI. Quarto website is inside Posit Netlify account. 
- Automatic builds are turned off for this repo on Netlify because we currently need to render with Quarto CLI in Gihub Action CI before publishing the results.
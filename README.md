# Vercel Action

## Introduction to Vercel
​Vercel is a cloud platform for **static sites** and **Serverless Functions** that fits perfectly with your workflow. It enables developers to host **Jamstack** websites and web services that **deploy instantly**, **scale automatically**, and requires **no supervision**, all with **no configuration**.

This action make a Vercel deployment with github actions. 

## Inputs

| Name              | Required | Default | Description                                                                                       |
|-------------------|:--------:|---------|---------------------------------------------------------------------------------------------------|
| vercel-token      |    [x]   |         | Vercel token. see https://vercel.com/account/tokens                                                                                  |
| github-comment    |    [ ]   |  false  | if you don't want to comment on pull request.                                                     |
| github-token      |    [ ]   |         | if you want to comment on pull request or commit. `${{ secrets.GITHUB_TOKEN }}` ([GitHub token docs](https://docs.github.com/en/actions/configuring-and-managing-workflows/authenticating-with-the-github_token))                                                         |
| vercel-project-id |    [x]   |         | ❗Vercel CLI 17+,The `name` property in vercel.json is deprecated (https://zeit.ink/5F)                  |
| vercel-org-id     |    [x]   |         | ❗Vercel CLI 17+,The `name` property in vercel.json is deprecated (https://zeit.ink/5F)                  |
| vercel-args       |    [ ]   |         | This is optional args for `vercel` cli. Example: `--prod`                                            |
| working-directory |    [ ]   |         | the working directory                                                                             |
| scope             |    [ ]   |         | If you are working in a team scope, you should set this value to your `team ID`.           
| alias-domains     |    [ ]   |         | You can assign a domain to this deployment. Please note that this domain must have been configured in the project. You can use pull request number via `{{PR_NUMBER}}` and branch via `{{BRANCH}}`.         
| vercel-project-name |  [ ]   |         | The name of the project; if absent we'll use the `vercel inspect` command to determine. [#27](https://github.com/amondnet/vercel-action/issues/27) & [#28](https://github.com/amondnet/vercel-action/issues/28)
## Outputs

### `preview-url`

The url of deployment preview.

### `preview-name`

The name of deployment name.

## How To Use

### Disable Vercel for GitHub

> The Vercel for GitHub integration automatically deploys your GitHub projects with Vercel, providing Preview Deployment URLs, and automatic Custom Domain updates.
[link](https://vercel.com/docs/v2/git-integrations)

We would like to to use `github actions` for build and deploy instead of `Vercel`. 

Set `github.enabled: false` in `vercel.json`, see example `vervel.json` file below:

```json
{
  "version": 2,
  "public": false,
  "github": {
    "enabled": false
  },
  "builds": [
    { "src": "./public/**", "use": "@now/static" }
  ],
  "routes": [
    { "src": "/(.*)", "dest": "public/$1" }
  ]
}

```
When set to false, `Vercel for GitHub` will not deploy the given project regardless of the GitHub app being installed.


### Project Linking

You should link a project via [Vercel CLI](https://vercel.com/download) in locally.

When running `vercel` in a directory for the first time, [Vercel CLI](https://vercel.com/download) needs to know which scope and Project you want to deploy your directory to. You can choose to either link an existing project or to create a new one.

> NOTE: Project linking requires at least version 17 of [Vercel CLI](https://vercel.com/download). If you have an earlier version, please [update](https://vercel.com/guides/updating-vercel-cli) to the latest version.

```bash
vercel
```

```bash
? Set up and deploy “~/web/my-lovely-project”? [Y/n] y
? Which scope do you want to deploy to? My Awesome Team
? Link to existing project? [y/N] y
? What’s the name of your existing project? my-lovely-project
🔗 Linked to awesome-team/my-lovely-project (created .vercel and added it to .gitignore)
```

Once set up, a new `.vercel` directory will be added to your directory. The `.vercel` directory contains both the organization(`vercel-org-id`) and project(`vercel-project-id`) id of your project.

```json
{"orgId":"example_org_id","projectId":"example_project_id"}
```

You can save both values in the secrets setting in your repository. Read the [Official documentation](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets) if you want further info on how secrets work on Github.

### Github Actions

* This is a complete `.github/workflow/deploy.yml` example.

Set the `vercel-project-id` and `vercel-org-id` you found above.

```yaml
name: deploy website
on: [pull_request]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: BetterNameX/vercel-action@master
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }} # Required
          github-token: ${{ secrets.GITHUB_TOKEN }} #Optional 
          vercel-args: '--prod' #Optional
          vercel-org-id: ${{ secrets.ORG_ID}}  #Required
          vercel-project-id: ${{ secrets.PROJECT_ID}} #Required 
          working-directory: ./sub-directory
```

### Alias Domains

You can assign a domain to this deployment. Please note that this domain must have been [configured](https://vercel.com/docs/v2/custom-domains#adding-a-domain) in the project.

If you want to assign domain to branch or pr, you should add [Wildcard Domain](https://vercel.com/docs/v2/custom-domains#wildcard-domains).

You can use pull request number via `{{PR_NUMBER}}` and branch via `{{BRANCH}}`

#### Example 

Wildcard Domains : *.angular.vercel-action.amond.dev

*Per Pull Request*

https://pr-{{PR_NUMBER}}.angular.vercel-action.amond.dev

- PR-1 -> https://pr-1.angular.vercel-action.amond.dev
- PR-2 -> https://pr-2.angular.vercel-action.amond.dev

*Per Branch*

https://{{BRANCH}}.angular.vercel-action.amond.dev

- develop -> https://develop.angular.vercel-action.amond.dev
- master -> https://master.angular.vercel-action.amond.dev
- master -> https://master.angular.vercel-action.amond.dev



See [.github/workflows/example-angular.yml](/.github/workflows/example-angular.yml) 

```yaml
name: deploy website
on: [pull_request]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: BetterNameX/vercel-action@master
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }} # Required
          github-token: ${{ secrets.GITHUB_TOKEN }} #Optional 
          vercel-args: '--prod' #Optional
          vercel-org-id: ${{ secrets.ORG_ID}}  #Required
          vercel-project-id: ${{ secrets.PROJECT_ID}} #Required 
          working-directory: ./sub-directory #Your Working Directory, Optional
          alias-domains: | #Optional
            staging.angular.vercel-action.amond.dev
            pr-{{PR_NUMBER}}.angular.vercel-action.amond.dev
```

# Package

```bash
npm run package
```

# Release

```bash
git add dist
git tag --new-release
```



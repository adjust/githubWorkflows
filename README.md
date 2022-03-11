# githubWorkflows

Shared work flows

## Introduction

In a decentralized repository structure we will have application code segregated into separate repositories. This means that the CI/CD and the tasks around them has to be repeated in all the repos.
Though it looks convenient initially, situation can quickly slip out of hand.

As time goes we will figure out this approach can be

- Tedious
- Error prone
- Non uniform
- Difficult to manage
- Difficult to enforce governance

In order to avoid this we can use reusable github work flows

### Advandages

- **Easy of application CI/CD**<br/>
  With few lines of configurations new applications can hit the road running by calling these workflows.
- **Reduce the lead time to containerize apps substantially**<br/>
- **Uniform process across organization**(more or less)<br/>
  One major nightmare when it comes to operating applications on an organization wide platform is to accommodate various different ways that apps use to build and deploy apps. Most cases this variance is unnecessary. This coupled with terminology mismatches will erect silos creating further barriers for communication. As a result teams paths will diverge from each other and lead to a point of no return.
- **Code governance**<br/>
  Centralized pipelines are the places where governance and policies regarding over all code hygiene, security etc can be enforced.
- **Flexibility to teams**<br/>
  However this frame work has to be modular enough to give advantage to apps to add or remove logic to the reusable workflow. This can be done by adding additional logic in the github workflows in the repositories.

### Limitations

Github reusable workflows is a relatively new feature which will need more time to stabilize. All these limitations apply (extracted from github)

1. Reusable workflows can't call other reusable workflows.
2. Reusable workflows stored within a private repository can only be used by workflows within the same repository.
3. Any environment variables set in an env context defined at the workflow level in the caller workflow are not propagated to the called workflow. For more information about the env context, see "Context and expression syntax for GitHub Actions."
4. The strategy property is not supported in any job that calls a reusable workflow.

Zooming in on Limitation #2,

- We cannot have a reusable workflow in a private repo.
- If we keep reusable workflow within a private repo only the workflows within that same repo can access it. This breaks the deal.
- For the reusable github workflow to be used at full potential it has to be a public repo.
- There is a middle path called "Internal repositories" where we can share the workflows selectively to organizational repos. But this feature is limited to enterprise customers.

**From the above observations its very essential to write shared workflows with extreme caution so that we dont leak any critical information into the workflow.
So this repo needs strict audit and changes has to be strictly reviewed.**

## Maintenance

- Collectively maintained by a guild
- Audited regularly with security team
- Changes has to be reviewed thouroughly

## Workflows

### buildScanPush

This workflow give features to

- Build containers
- Security scan the images
- Push images to repository

Scan can be configured to ignore if there are CRITICAL flaws. This is an example of how we can enforce policies at high level. We can add code hygiene, slack comms etc etc

Expects following configs passed from the caller workflow:

| Parameter               | Description                                              | Default            | Example                                            | Required |
| ----------------------- | -------------------------------------------------------- | ------------------ | -------------------------------------------------- | -------- |
| `image_name`            | Image name                                               |                    |                                                    | yes      |
| `tags`                  | Image tags, for multiple tags use string delimited by \n | `""`               | `test1\ntest2`                                     | No       |
| `registry`              | Container image registry                                 |                    | `1234567789012.dkr.ecr.eu-central-1.amazonaws.com` | yes      |
| `build_args`            | Build args, for multiple tags use string delimited by \n | `cloud.adjust.com` | `arg0=val1,val2\narg1=test1\narg2=test2`           | No       |
| `region`                | Region of registry                                       | `eu-central-1`     | `eu-central-1`                                     | No       |
| `dockerfile_path`       | Docker context where Dockerfiles are located             | `.`                | `./dockerfile_dir`                                 | No       |
| `exit_on_scan_fail`     | Don't push if CRITICAL vulnerabilities are detected      | `0`                |                                                    | No       |
| `AWS_ACCESS_KEY_ID`     | AWS credential for container registry                    |                    |                                                    | Yes      |
| `AWS_SECRET_ACCESS_KEY` | AWS credential for container registry                    |                    |                                                    | Yes      |
| `TEAM_GITHUB_TOKEN`     | GITHUB token for github access while builds              |                    |                                                    | No       |

### deployToK8s

This workflow give features to deploy your application to cluster of choice

Expects following configs passed from the caller workflow:

| Parameter               | Description                                        | Default        | Example                                                         | Required |
| ----------------------- | -------------------------------------------------- | -------------- | --------------------------------------------------------------- | -------- |
| `image_url`             | Image url repository/image:tag                     |                | `1234567789012.dkr.ecr.eu-central-1.amazonaws.com/test:tag`     | yes      |
| `migrate_job`           | Some dashboard apps need migrate jobs as pre start | `""`           | `dashboard-db-migrate`                                          | No       |
| `deployments`           | Deployment names, can give multiple names          | `""`           | `dashboard dashboard-sidekiq dashboard-radlis dashboard-kafkis` | yes      |
| `region`                | Region to deploy to                                | `eu-central-1` | `eu-central-1`                                                  | No       |
| `cluster`               | k8s cluster to deploy the application to           | `""`           | `qa-eks`                                                        | Yes      |
| `namespace`             | k8s namesapce to deploy the application to         | `""`           | `abcd-1234`                                                     | Yes      |
| `url`                   | URL where app listens                              |                | `https://api-abcd-1234.cloud.adjust.com/dashboard/heartbeat`    | Yes      |
| `AWS_ACCESS_KEY_ID`     | AWS credential for container registry              |                |                                                                 | Yes      |
| `AWS_SECRET_ACCESS_KEY` | AWS credential for container registry              |                |                                                                 | Yes      |
| `TEAM_GITHUB_TOKEN`     | GITHUB token for github access while builds        |                |                                                                 | Yes      |

### datacenterDeploy

This workflow give features to deploy a datacenter

Expects following configs passed from the caller workflow:

| Parameter               | Description                                               | Default                                | Example                                            | Required |
| ----------------------- | --------------------------------------------------------- | -------------------------------------- | -------------------------------------------------- | -------- |
| `dcname`                | Name of the datacenter                                    |                                        | `qa`                                               | yes      |
| `goal`                  | Make goal to execute                                      | `""`                                   | `dashboard-db-migrate`                             | yes      |
| `region`                | Region where datacenter is deployed                       | `eu-central-1`                         | `eu-central-1`                                     | no       |
| `user`                  | User deploying the datacenter, for tagging and accounting | `""`                                   | `qa`                                               | yes      |
| `vault`                 | Vault address where secrets/certs reside                  | `""`                                   | `http://vault.cloud.com`                           | Yes      |
| `registry`              | Container image registry                                  | `""`                                   | `1234567789012.dkr.ecr.eu-central-1.amazonaws.com` | Yes      |
| `runner`                | Docker runner image used to spin up the deploy runner     | `platform-actions-builder:ubuntu20_04` |                                                    | Yes      |
| `AWS_ACCESS_KEY_ID`     | AWS credential for container registry                     |                                        |                                                    | Yes      |
| `AWS_SECRET_ACCESS_KEY` | AWS credential for container registry                     |                                        |                                                    | Yes      |
| `TEAM_GITHUB_TOKEN`     | GITHUB token for github access while builds               |                                        |                                                    | Yes      |
| `VAULT_TOKEN`           | Vault token to authenticate against vault                 |                                        |                                                    |          |

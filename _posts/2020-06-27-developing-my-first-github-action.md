---
title: Developing my first GitHub Action 
key: 20200627
author: Venura Athukorala
tags: github github-actions azure security devops
---

Github actions have already shown a lot of promise and some distance to travel before it can stand next to its big brother 'Azure DevOps'. When I say 'big brother', they're both technically Microsoft Products now. 

Azure DevOps for historical reasons was tagged as Azure native and Azure only. But, it's not really an Azure only and I'm aware of organisations using Azure DevOps to manage their infrastructure. Don't believe me... time to take the red pill and read about the [AWS Toolkit for Microsoft Azure DevOps](https://docs.aws.amazon.com/vsts/latest/userguide/welcome.html)

Common sense prevailing, it's right to market GitHub as the true multi-cloud champion instead of Azure DevOps (previously known as Microsoft VSTS,TFS). Which is what Microsoft seems to be doing with the addition of number features from areas such as Azure Pipelines and Visual Studio Codespaces to make the product more robust and enterprise friendly. 

In this post I'm going to explain how to create a custom action for Azure, using my experience building an action as a part of [https://githubhackathon.com](https://githubhackathon.com)

> I'm in the [Winners List](https://docs.google.com/spreadsheets/d/1YL6mjJXGt3-75GejQCubsOvWwtYcGaqbJA7msnsh7Tg) :)
> - [My Action repository](https://github.com/marketplace/actions/manage-nsg)
> - [My Action in Marketplace](https://github.com/marketplace/actions/manage-nsg)

Actions documentation is pretty concise and helps you to get started quickly. Below links cover the basics for you to get started;

- [Getting started with GitHub Actions](https://help.github.com/en/actions/getting-started-with-github-actions)
- [Creating GitHub Actions](https://help.github.com/en/actions/creating-actions/about-actions)

There are two types of actions "JavaScript and Docker Container". While JavaScript directly runs on the runner, allowing for faster execution, the Docker Container allows for a consistent and reliable unit without having to worry about tools and dependencies.

I went for the reliability and decided to live with the extra time taken to build the container. 

## Setup

To make 'GitHub Actions' ready, you need to tick off few things as a part of your github repository;


1. A `LICENSE` - A license that permits others to use the action. I've gone wit MIT
2. A `README.md` - This will be used as the marketplace description. Try to provide sufficient information on how to use the action with examples. Here's mine: https://github.com/marketplace/actions/manage-nsg
3. Metadata file - `action.yml`
 * inputs - input variables
 * outputs - output variables which can be used later in the workflow
 * branding - required for publishing to the marketplace
 * runs - `using: 'docker'` with `image: 'Dockerfile'` and the inputs set as `args: ...` 
4. An `entrypoint.sh` expressing what you need to run on the container. 
5. A Dockerfile - [[Doc](https://help.github.com/en/actions/creating-actions/dockerfile-support-for-github-actions)]

For my first attempt tried to start with a vanilla alpine and quickly realised that I'm trying to re-invent the wheel. Best starting ground for azure is to use the official azure cli container. 

Below is the working Dockerfile content of my action;

```bash
# Using latest might cause issues with breaking changes.
# FROM mcr.microsoft.com/azure-cli:latest

# https://hub.docker.com/_/microsoft-azure-cli
# https://mcr.microsoft.com/v2/azure-cli/tags/list
FROM mcr.microsoft.com/azure-cli:2.8.0

# Copies your code file from your action repository to the filesystem path `/` of the container.
COPY entrypoint.sh /entrypoint.sh

# Set the missing exec permission, just in case if you're on on a *nix. 
RUN chmod +x ./entrypoint.sh

# Enable dig to find the runner's public IP
RUN apk update && apk add --no-cache bind-tools && rm -rf /var/cache/apk/*

# Code file to execute when the docker container starts up (`entrypoint.sh`)
ENTRYPOINT ["/entrypoint.sh"]
```

* Consideration #1

Don't use `FROM mcr.microsoft.com/azure-cli:latest`. Versions change pretty fast and you wouldn't want your workflow to break due to uncontrollable changes. Refer to the [list of tags](https://mcr.microsoft.com/v2/azure-cli/tags/list) and use the latest at the point of creation and rollout updates overtime. I've started with `2.2.0` and moved to `2.8.0` at the point current update to the post. 

E.g. `FROM mcr.microsoft.com/azure-cli:2.8.0`

*  Consideration #2

Make sure your `entrypiont.sh` file is executable. Use `RUN chmod +x ./entrypoint.sh` in your `Dockerfile`


## What

My action is capable of two things. 

1. Create a temporary Azure NSG Rule allowing a given port of the runner's public IP to access resources behind resource protected by an Azure NSG. 

E.g. Deploy to a protected Web App running on an ASE or an Azure VM. 

2. Remove the temporary rule after deployment is completed. 

Let's look at what the `entrypoint.sh` does. Comments should tell you all about it!

```bash
#!/bin/sh -l

# Ensure the workflow fails on error
set -e

_azure_credentails=$1
_rule_priority_start=$2
_rule_priority_end=$(($2+$3))
_rule_port=$4
_rule_id_for_removal=$5
_rule_nsg_resource_group=$6
_rule_nsg=$7
_rule_public_ip=$(dig +short myip.opendns.com @resolver1.opendns.com) # Get runner's public IP


# break the azure credientials JSON
_client_id=$(echo $_azure_credentails | jq -r '.clientId')
_client_secret=$(echo $_azure_credentails | jq -r '.clientSecret')
_tenant_id=$(echo $_azure_credentails | jq -r '.tenantId')
_subscription_id=$(echo $_azure_credentails | jq -r '.subscriptionId')

# Login to azure using service principal
az login --service-principal -u $_client_id -p $_client_secret --tenant $_tenant_id

# Select the subscription
az account set --subscription $_subscription_id

if [ $_rule_id_for_removal ]
then
    # removing the rule
    echo "Removing rule $_rule_id_for_removal"
    az network nsg rule delete -g $_rule_nsg_resource_group --nsg-name $_rule_nsg -n $_rule_id_for_removal
    echo "::set-output name=rule_name::$_rule_id_for_removal"
else
    # adding the rule
    echo _rule_port: $_rule_port
    echo _rule_priority_start: $_rule_priority_start
    echo _rule_priority_end: $_rule_priority_end

    _rule_priority=$(shuf -i $_rule_priority_start-$_rule_priority_end -n 1)
    _rule_name=manage-nsg-github-actions-$_rule_priority

    echo "Adding rule.... $_rule_name"
    az network nsg rule create -g $_rule_nsg_resource_group --nsg-name $_rule_nsg \
				-n $_rule_name --priority $_rule_priority \
				--source-address-prefixes $_rule_public_ip --source-port-ranges '*' \
				--destination-address-prefixes '*' --destination-port-ranges '*' \
				--access Allow --protocol '*' \
				--description 'Allow from IP address of github actions hosted runner temporarily'
    # output
    echo "::set-output name=rule_name::$_rule_name"
fi

```

## Publish

- Publishing is easy. Go to the releases and tick the box to "Publish this Action to the GitHub Marketplace"

- Define a tag which allows the users to define which version of the action to use. 

- Select your categories. 

- Give a Title and a Description for the release. 

- Hit "Publish Release"

- Make sure to tag the unstable releases as "This is a pre-release" to let the users know that it's not ready for consumption. 

![Publish view](https://whatcloud.xyz/assets/publish-github-action.png "Publish view")

## Test

Testing is easy. All you need is a workflow that runs `on: push` for the relevant branches and test the action.

Note: Instead of using a specific release I'm using the branch to ensure that I'm testing the latest code on the branch. `uses: venura9/manage-nsg@master`


```yaml
name: run_test_master

on:
  push:
    branches:
      - master

jobs:
  test:
    runs-on: ubuntu-latest
    name: Run GitHub Action Tests
    steps:

      - name: dig +short myip.opendns.com @resolver1.opendns.com
        run: dig +short myip.opendns.com @resolver1.opendns.com

      - name: Add NSG Rule
        uses: venura9/manage-nsg@master
        id: rule
        with:
          azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
          rule-nsg-resource-group-name: ManageNsg
          rule-nsg-name: ManageNsg

      - name: Print Created NSG Rule Name
        run: echo "Rule Name ${{ steps.rule.outputs.rule_name }}"

      - name: Remove NSG Rule
        uses: venura9/manage-nsg@master
        with:
          azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
          rule-id-for-removal: ${{ steps.rule.outputs.rule_name }}
          rule-nsg-resource-group-name: ManageNsg
          rule-nsg-name: ManageNsg
```

### Local Actions
You can test the action even before publishing using the local actions. This feature allows for private actions that are specific to a repository. 

```yaml
jobs:
  my_first_job:
    steps:
      - name: Check out repository
        uses: actions/checkout@v2
      - name: Use local my-action
        uses: ./.github/actions/my-action
```

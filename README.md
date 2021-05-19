# packer-w2019

Blog Post

<https://alexholliz.github.io/aws/packer/windows/2021/05/18/building-a-windows-server-ami-with-packer.html>

This repo takes the instructions for building packer as described in that blog post, and to it, adds Github Actions to automatically build a base Windows Server 2019 AMI.

## Prereqs

Github Repository Secrets for your: 
* AWS Access Key ID
* AWS Secret Access Key
* VPC ID
* Subnet ID
* AWS Region

## Actions

The Actions file is pretty dang simple, set a few environment variables to use your repository secrets, and use the Packer actions from Hashicorp: <https://github.com/marketplace/actions/packer-github-actions>

Below is the main.yml for the Actions workflow, but the breakdown is:

- Run on: Push and Manual start

- Job called "Packer"

1. Sync code from repo
2. Do a packer validate with secrets as environment variables
3. Do a packer build with secrets as environment variables

```
# This is a basic workflow to help you get started with Actions

name: Packer

# Controls when the action will run. 
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  packer:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2

      # validate templates
      - name: Validate Template
        uses: hashicorp/packer-github-actions@master
        with:
          command: validate
          arguments: -syntax-only
          target: packer.pkr.hcl
        env:
          PKR_VAR_aws_region: ${{ secrets.AWS_REGION }}
          PKR_VAR_vpc_id: ${{ secrets.VPC_ID }}
          PKR_VAR_subnet_id: ${{ secrets.SUBNET_ID }}
          PKR_VAR_AKID: ${{ secrets.PACKER_IAM_AKID }}
          PKR_VAR_SKEY: ${{ secrets.PACKER_IAM_SKEY }}

      # build artifact
      - name: Build Artifact
        uses: hashicorp/packer-github-actions@master
        with:
          command: build
          arguments: "-color=false -on-error=abort"
          target: packer.pkr.hcl
        env:
          PACKER_LOG: 1
          PKR_VAR_aws_region: ${{ secrets.AWS_REGION }}
          PKR_VAR_vpc_id: ${{ secrets.VPC_ID }}
          PKR_VAR_subnet_id: ${{ secrets.SUBNET_ID }}
          PKR_VAR_AKID: ${{ secrets.PACKER_IAM_AKID }}
          PKR_VAR_SKEY: ${{ secrets.PACKER_IAM_SKEY }}
```
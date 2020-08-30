# Simple Terraform Tutorial Workshop for AWS Cloud

In this workshop, we'll create an AWS S3 bucket with Terraform.  For simplicity, we'll use local storage for the statefile. Local storage should only be used for light testing. For real-world usage, you should use a remote backend.

## Summary

1. Install Terraform
2. Configure AWS
3. Deploy Infrastructure

## Install Terraform

Here is the official [Terraform Download page](https://www.terraform.io/downloads.html).  You can follow those instructions. Just make sure to put the terraform executable in a folder that is found in the PATH. IE: `which terraform` should show the location of terraform.

Another good way to install terraform is with [tfenv](https://github.com/tfutils/tfenv) - Terraform version manager. This allows you to quickly switch between different versions of Terraform. Here are instructions for tfenv:

    git clone https://github.com/tfutils/tfenv.git ~/.tfenv
    echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.bash_profile

Open a new terminal and then run:

    tfenv install 0.13.1

## Configure AWS

Configure AWS so Terraform can connect to it. The recommended way is to:

1. set up the `~/.aws/config` and `~/.aws/credentials` files
2. set up your `AWS_PROFILE` and `AWS_REGION` environment variables

## Example Config Setup

~/.aws/config

    [dev]
    output = json
    region = us-west-2

~/.aws/credentials

    [dev]
    aws_access_key_id = REPLACE_ME
    aws_secret_access_key = REPLACE_ME

In your `~/.bashrc` or `~/.profile`, use this line to set the `AWS_PROFILE` and `AWS_REGION` environment variables:

    export AWS_PROFILE=dev
    export AWS_REGION=`aws configure get region` # to match what's in ~/.aws/config

The reason we have to configure `AWS_REGION` also, is because Terraform doesn't seem to use the `~/.aws/config` setting, but will use the `AWS_REGION`.

## Test AWS Setup

Here are some useful commands to test that the AWS CLI is working:

    aws sts get-caller-identity
    aws s3 ls

More Docs: [AWS CLI Docs](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)

## Deploy

    terraform init
    terraform apply

Example with output:

    $ terraform apply

    An execution plan has been generated and is shown below.
    Resource actions are indicated with the following symbols:
      + create

    Terraform will perform the following actions:

      # aws_s3_bucket.this will be created
      + resource "aws_s3_bucket" "this" {
          + acceleration_status         = (known after apply)
          + acl                         = "private"
          + arn                         = (known after apply)
          + bucket                      = (known after apply)
          + bucket_domain_name          = (known after apply)
          + bucket_regional_domain_name = (known after apply)
          + force_destroy               = false
          + hosted_zone_id              = (known after apply)
          + id                          = (known after apply)
          + region                      = (known after apply)
          + request_payer               = (known after apply)
          + website_domain              = (known after apply)
          + website_endpoint            = (known after apply)

          + versioning {
              + enabled    = (known after apply)
              + mfa_delete = (known after apply)
            }
        }

    Plan: 1 to add, 0 to change, 0 to destroy.

    Do you want to perform these actions?
      Terraform will perform the actions described above.
      Only 'yes' will be accepted to approve.

      Enter a value:

You're prompted to confirm. Type `yes` and press enter:

      Enter a value: yes

    aws_s3_bucket.this: Creating...
    aws_s3_bucket.this: Creation complete after 1s [id=terraform-20200802233942217700000001]

    Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

    Outputs:

    name = terraform-20200802233942217700000001
    $

You can see a bucket was created.

## Explore

It is useful to explore some of the files that Terraform created. This helps understand what Terraform does and how it works.

The previous commands created a `.terraform` folder and a `terraform.tfstate` file.  They'll look something like this:

    .terraform
    ├── plugins
    │   └── linux_amd64
    │       ├── lock.json
    │       └── terraform-provider-aws_v3.0.0_x5
    └── terraform.tfstate

When you ran `terraform init`, terraform evaluated your Terraform code and detected that it needed to download the aws plugin. This is how Terraform knows how to create the AWS resources.

Then when you ran `terraform apply` it applied the changes and created the S3 Bucket. Since we did not configure a backend.tf, the state information is stored locally in the `terraform.tfstate`. Go ahead and check out the contents of the file. Here's a `cat` command with `jq` to check out the file:

    cat terraform.tfstate | jq

Here's also relevant part of the statefile.

```json
{
  "version": 4,
  "terraform_version": "0.12.29",
  "serial": 20,
  "lineage": "96fde5c0-95e8-5edd-df8c-005c09e8a84a",
  "resources": [
    {
      "mode": "managed",
      "type": "aws_s3_bucket",
      "name": "this",
      "provider": "provider.aws",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "acceleration_status": "",
            "acl": "private",
            "arn": "arn:aws:s3:::terraform-20200802233942217700000001",
            "bucket": "terraform-20200802233942217700000001",
            "bucket_domain_name": "terraform-20200802233942217700000001.s3.amazonaws.com",
# ...
```

This is how terraform keeps track of what has been created and how to manage the resources. As such, this is a crucial file.  In real-world usage, the statefile is stored in a remote backend like an S3 bucket and versioned.

## Cleanup

Let's clean up and delete the resources now.

    terraform destroy

You'll be prompted. Type `yes` to confirm.

Take another look at the `terraform.tfstate` file.

    cat terraform.tfstate | jq

You'll see something like this:

```json
{
  "version": 4,
  "terraform_version": "0.12.29",
  "serial": 22,
  "lineage": "96fde5c0-95e8-5edd-df8c-005c09e8a84a",
  "outputs": {},
  "resources": []
}
```

You can see that the statefile also reflects that there are no resources.

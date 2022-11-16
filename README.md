<!-- markdownlint-configure-file {

  "MD013": {

    "code_blocks": false,

    "tables": false

  },

  "MD033": false,

  "MD041": false

} -->

<div align="left">

# ⚡️ Lambda Layers

This repo contains a CloudFormation YAML script that creates a Linux EC2 Instance, builds and publish an AWS Python Lambda Layer. 
</div>

## What is a Lambda Layer?

![Layers](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/86hc1c4myw9jw9enwqtc.gif)


Read more about AWS Lambda Layers [here](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html).

## Deployment

### *Step 1: Deploy using AWS CLI*

You can deploy the CloudFormation Stack using AWS CLI. For more information about aws cloudformation commands visit [here](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/index.html)


## AWS CLI Command
```bash
aws cloudformation deploy --template-file <path_to_file> --stack-name <stack_name> --capabilities CAPABILITY_NAMED_IAM

```



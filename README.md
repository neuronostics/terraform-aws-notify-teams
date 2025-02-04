# AWS Notify Microsoft Teams Terraform module

## Neuronostics fork

This forks the upstream terraform module to add support for sending messages to a Microsoft Teams channel via an incoming webhook. The upstream module misses some appropriate `Content-Type` headers to successfully send the message. This fork adds those headers.

## Original README

This module creates an SNS topic (or uses an existing one) and an AWS Lambda function that sends notifications to Microsoft Teams using the [Incoming Webhook Connector](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook).

Start by setting up an [incoming webhook](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook#add-an-incoming-webhook-to-a-teams-channel) in your Microsoft Teams channel.

Doing serverless with Terraform? Check out [serverless.tf framework](https://serverless.tf), which aims to simplify all operations when working with the serverless in Terraform.


## Terraform versions

Terraform 0.13. Pin module version to `~> v4.0`. Submit pull-requests to `master` branch.

Terraform 0.12. Pin module version to `3.5.0` (or older). Submit pull-requests to `terraform012` branch.

Terraform 0.11. Pin module version to `~> v1.0`.

## Features

- [x] AWS Lambda runtime Python 3.9
- [x] Create new SNS topic or use existing one
- [ ] Support plaintext and encrypted version of Teams webhook URL
- [ ] Most of Teams message options are customizable
- [x] Support different types of SNS messages:
  - [x] AWS CloudWatch Alarms
  - [x] AWS CloudWatch LogMetrics Alarms
  - [ ] [Send pull-request to add support of other message types](https://github.com/teamclairvoyant/terraform-aws-notify-teams/pulls)
- [ ] Local pytest driven testing of the lambda to a Teams sandbox channel

## Usage

```hcl
module "notify_teams" {
  source  = "git::https://github.com/teamclairvoyant/terraform-aws-notify-teams.git?ref=tags/<LATEST_TAG>"

  sns_topic_name = "teams-topic"

  teams_webhook_url = "https://outlook.office.com/webhook/AAA@BBB/IncomingWebhook/CCC/DDD"
}
```

## Upgrade from 2.0 to 3.0

Version 3 uses [Terraform AWS Lambda module](https://github.com/terraform-aws-modules/terraform-aws-lambda) to handle most of heavy-lifting related to Lambda packaging, roles, and permissions, while maintaining the same interface for the user of this module after many of resources will be recreated.

## Using with Terraform Cloud Agents

[Terraform Cloud Agents](https://www.terraform.io/docs/cloud/workspaces/agent.html) are a paid feature, available as part of the Terraform Cloud for Business upgrade package.

This module requires Python 3.8. You can customize [tfc-agent](https://hub.docker.com/r/hashicorp/tfc-agent) to include Python using this sample `Dockerfile`:

```
FROM hashicorp/tfc-agent:latest
RUN apt-get -y update && apt-get -y install python3.8 python3-pip
ENTRYPOINT ["/bin/tfc-agent"]
```

## Use existing SNS topic or create new

If you want to subscribe the AWS Lambda Function created by this module to an existing SNS topic you should specify `create_sns_topic = false` as an argument and specify the name of existing SNS topic name in `sns_topic_name`.


## Testing with pytest

To run the tests:

1. Set up a dedicated Teams channel as a test sandbox with it's own webhook. See [add an incoming webhook to a Teams channel](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook#add-an-incoming-webhook-to-a-teams-channel) for details.
2. Make a copy of the sample pytest configuration and edit as needed.

        cp functions/pytest.ini.sample functions/pytest.ini

3. Run the tests:

        pytest functions/notify_teams_test.py

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Requirements

| Name | Version |
|------|---------|
| terraform | >= 0.13.0 |
| aws | >= 2.35 |

## Providers

| Name | Version |
|------|---------|
| aws | >= 2.35 |

## Modules

| Name | Source | Version |
|------|--------|---------|
| lambda | terraform-aws-modules/lambda/aws | 1.28.0 |

## Resources

| Name |
|------|
| [aws_caller_identity](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/caller_identity) |
| [aws_cloudwatch_log_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudwatch_log_group) |
| [aws_iam_policy_document](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_policy_document) |
| [aws_region](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/region) |
| [aws_sns_topic](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/sns_topic) |
| [aws_sns_topic_subscription](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/sns_topic_subscription) |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| cloudwatch\_log\_group\_kms\_key\_id | The ARN of the KMS Key to use when encrypting log data for Lambda | `string` | `null` | no |
| cloudwatch\_log\_group\_retention\_in\_days | Specifies the number of days you want to retain log events in log group for Lambda. | `number` | `0` | no |
| cloudwatch\_log\_group\_tags | Additional tags for the Cloudwatch log group | `map(string)` | `{}` | no |
| create | Whether to create all resources | `bool` | `true` | no |
| create\_sns\_topic | Whether to create new SNS topic | `bool` | `true` | no |
| iam\_role\_boundary\_policy\_arn | The ARN of the policy that is used to set the permissions boundary for the role | `string` | `null` | no |
| iam\_role\_name\_prefix | A unique role name beginning with the specified prefix | `string` | `"lambda"` | no |
| iam\_role\_tags | Additional tags for the IAM role | `map(string)` | `{}` | no |
| kms\_key\_arn | ARN of the KMS key used for decrypting teams webhook url | `string` | `""` | no |
| lambda\_description | The description of the Lambda function | `string` | `null` | no |
| lambda\_function\_name | The name of the Lambda function to create | `string` | `"notify_teams"` | no |
| lambda\_function\_s3\_bucket | S3 bucket to store artifacts | `string` | `null` | no |
| lambda\_function\_store\_on\_s3 | Whether to store produced artifacts on S3 or locally. | `bool` | `false` | no |
| lambda\_function\_tags | Additional tags for the Lambda function | `map(string)` | `{}` | no |
| lambda\_function\_vpc\_security\_group\_ids | List of security group ids when Lambda Function should run in the VPC. | `list(string)` | `null` | no |
| lambda\_function\_vpc\_subnet\_ids | List of subnet ids when Lambda Function should run in the VPC. Usually private or intra subnets. | `list(string)` | `null` | no |
| lambda\_role | IAM role attached to the Lambda Function.  If this is set then a role will not be created for you. | `string` | `""` | no |
| log\_events | Boolean flag to enabled/disable logging of incoming events | `bool` | `false` | no |
| reserved\_concurrent\_executions | The amount of reserved concurrent executions for this lambda function. A value of 0 disables lambda from being triggered and -1 removes any concurrency limitations | `number` | `-1` | no |
| teams\_webhook\_url | The URL of Microsoft Teams webhook | `string` | n/a | yes |
| sns\_topic\_kms\_key\_id | ARN of the KMS key used for enabling SSE on the topic | `string` | `""` | no |
| sns\_topic\_name | The name of the SNS topic to create | `string` | n/a | yes |
| sns\_topic\_tags | Additional tags for the SNS topic | `map(string)` | `{}` | no |
| subscription\_filter\_policy | (Optional) A valid filter policy that will be used in the subscription to filter messages seen by the target resource. | `string` | `null` | no |
| tags | A map of tags to add to all resources | `map(string)` | `{}` | no |

## Outputs

| Name | Description |
|------|-------------|
| lambda\_cloudwatch\_log\_group\_arn | The Amazon Resource Name (ARN) specifying the log group |
| lambda\_iam\_role\_arn | The ARN of the IAM role used by Lambda function |
| lambda\_iam\_role\_name | The name of the IAM role used by Lambda function |
| notify\_teams\_lambda\_function\_arn | The ARN of the Lambda function |
| notify\_teams\_lambda\_function\_invoke\_arn | The ARN to be used for invoking Lambda function from API Gateway |
| notify\_teams\_lambda\_function\_last\_modified | The date Lambda function was last modified |
| notify\_teams\_lambda\_function\_name | The name of the Lambda function |
| notify\_teams\_lambda\_function\_version | Latest published version of your Lambda function |
| this\_teams\_topic\_arn | The ARN of the SNS topic from which messages will be sent to Teams |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->

## Authors

Module managed by [Anton Babenko](https://github.com/antonbabenko).

## License

Apache 2 Licensed. See LICENSE for full details.

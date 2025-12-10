# Helix DataDog Feeder

> Service that subscribes to CloudWatch logs for Helix services and pushes them to DataDog.

## Status
[![codecov](https://img.shields.io/codecov/c/github/adobe/helix-datadog-feeder.svg)](https://codecov.io/gh/adobe/helix-datadog-feeder)
[![CircleCI](https://img.shields.io/circleci/project/github/adobe/helix-datadog-feeder.svg)](https://circleci.com/gh/adobe/helix-datadog-feeder)
[![GitHub license](https://img.shields.io/github/license/adobe/helix-datadog-feeder.svg)](https://github.com/adobe/helix-datadog-feeder/blob/main/LICENSE.txt)
[![GitHub issues](https://img.shields.io/github/issues/adobe/helix-datadog-feeder.svg)](https://github.com/adobe/helix-datadog-feeder/issues)
[![LGTM Code Quality Grade: JavaScript](https://img.shields.io/lgtm/grade/javascript/g/adobe/helix-datadog-feeder.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/adobe/helix-datadog-feeder)
[![semantic-release](https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg)](https://github.com/semantic-release/semantic-release)

## Installation

The AWS Console has an issue where a subscription filter can not be added with a specific version or alias, we therefore recommend to use the AWS CLI.

Given the service you want to push logs into DataDog for, e.g. `helix-services--my-service`, use the following command:

```
$ AWS_REGION=...; AWS_ACCOUNT_ID=...; aws logs put-subscription-filter \
  --log-group-name /aws/lambda/helix-services--my-service \
  --filter-name helix-datadog-feeder \
  --filter-pattern '[timestamp=*Z, request_id="*-*", event]' \
  --destination-arn "arn:aws:lambda:${AWS_REGION}:${AWS_ACCOUNT_ID}:function:helix3--datadog-feeder:v1"
```

You can filter log events sent by level as follows:
```
  --filter-pattern '[timestamp=*Z, request_id="*-*", level=%WARN|ERROR%, event]'
```
this will invoke the feeder only for WARN and ERROR messages.

If you get an error that CloudWatch is not allowed to execute your function, add the following permission:
```
$ AWS_REGION=...; AWS_ACCOUNT_ID=...; aws lambda add-permission \
    --function-name "arn:aws:lambda:${AWS_REGION}:${AWS_ACCOUNT_ID}:function:helix3--datadog-feeder:v1" \
    --statement-id 'CloudWatchInvokeDataDog' \
    --principal 'logs.amazonaws.com' \
    --action 'lambda:InvokeFunction' \
    --source-arn "arn:aws:logs:${AWS_REGION}:${AWS_ACCOUNT_ID}:log-group:/aws/lambda/helix-services--my-service:*" \
    --source-account "${AWS_ACCOUNT_ID}"
```
If there are multiple services you want to add this subscription filter to, it is easier to replace the source arn
with a more generic expression:
```
    --source-arn "arn:aws:logs:${AWS_REGION}:${AWS_ACCOUNT_ID}:log-group:/aws/lambda/*:*"
```

The service uses the following environment variables:

| Name  | Description  | Required | Default |
|:------|:-------------|:---------|:--------|
| DATADOG_API_KEY | DataDog API Key | Yes | - |
| DATADOG_API_URL | DataDog Ingestion Base URL | No | https://http-intake.logs.datadoghq.com |
| DATADOG_LOG_LEVEL | Log level | No | info |

If delivery to DataDog fails, the service will send the unprocessed messages to the AWS SQS queue named `helix-datadog-feeder-dlq`.

## Development

### Deploying Helix DataDog Feeder

All commits to main that pass the testing will be deployed automatically. All commits to branches that will pass the testing will get commited as `helix3--datadog-feeder@ci<num>` and tagged with the CI build number.

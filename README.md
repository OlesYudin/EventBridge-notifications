# EventBridge-notifications

In this repository you can find useful AWS EventBridge patterns.

## Track login events to AWS Console through IAM User

To track login attempts to an AWS account using a specific IAM User, you can use AWS EventBridge.

> Note! [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-aws-console-sign-in-events.html "AWS CloudTrail") logs sign-in events using the root user in the `us-east-1` AWS Region. Login attempts using an IAM User are logged in the trail of the region where the user is attempting to sign in (for example, a login attempt in the Ohio AWS Region will be logged in CloudTrail in the `us-east-2` AWS Region).

In this example, the AWS EventBridge rule will trigger if there is an attempt to log in to an AWS account using the `UserDeveloper` IAM User.

```json
{
  "detail-type": ["AWS Console Sign In via CloudTrail"],
  "source": ["aws.signin"],
  "detail": {
    "eventSource": ["signin.amazonaws.com"],
    "eventName": ["ConsoleLogin"],
    "userIdentity": {
      "type": ["IAMUser"],
      "userName": ["UserDeveloper"]
    }
  }
}
```

## Track login events to AWS Console through AWS SSO Group

When using AWS Organization, you can track login attempts to an AWS account using a specific Permission Set. In this example, the AWS EventBridge rule will trigger if someone tries to log in to an AWS account using the `Administrator-***` group.

```json
{
  "detail-type": ["AWS Console Sign In via CloudTrail"],
  "source": ["aws.signin"],
  "detail": {
    "eventSource": ["signin.amazonaws.com"],
    "eventName": ["ConsoleLogin"],
    "userIdentity": {
      "type": ["AssumedRole"],
      "arn": [
        {
          "wildcard": "arn:aws:sts::<accound_id>:assumed-role/AWSReservedSSO_Administrator*/*"
        }
      ]
    }
  }
}
```

## Track attempts to delete CloudFormation stack

To monitor attempts to delete a CloudFormation stack, you can use the following AWS EventBridge rule pattern. In this example, AWS EventBridge will trigger if someone tries to delete a stack with the following names:

- `MyInfrastructure`;
- `MySecurityDev` or `MySecurityProd` - because the MySecurity is followed by an `*` (asterisk);
- `ImportantStackTest`, `NewImportantStackDev`, `NewImportantStackProd` - because ImportantStack is surrounded by `*` (asterisks).

```json
{
  "detail-type": ["CloudFormation Stack Status Change"],
  "resources": [
    {
      "wildcard": "arn:aws:cloudformation:*:*:stack/MyInfrastructure/*"
    },
    {
      "wildcard": "arn:aws:cloudformation:*:*:stack/MySecurity*/*"
    },
    {
      "wildcard": "arn:aws:cloudformation:*:*:stack/*ImportantStack*/*"
    }
  ],
  "source": ["aws.cloudformation"],
  "detail": {
    "status-details": {
      "status": ["DELETE_COMPLETE", "DELETE_FAILED", "DELETE_IN_PROGRESS"]
    }
  }
}
```

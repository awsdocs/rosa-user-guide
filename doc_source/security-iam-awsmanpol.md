# AWS managed IAM policies for ROSA<a name="security-iam-awsmanpol"></a>

To get started using IAM policies quickly, consider using AWS managed policies when adding permissions for users, groups, and roles\. If you use AWS managed policies, you don’t need to write your own policies\. It takes time and expertise to [create IAM customer managed policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create-console.html) that provide your team with the permissions that they need while still maintaining security best practice\. AWS managed policies cover common use cases, are available in your AWS account, and are quick to set up\. For more information about AWS managed policies, see [AWS managed policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies) in the * IAM User Guide*\.

 AWS services maintain and update AWS managed policies\. You can’t change the permissions in AWS managed policies\. AWS services occasionally add additional permissions to an AWS managed policy to support new features\. This type of update affects all the identities \(users, groups, and roles\) where the policy is attached\. Services are most likely to update an AWS managed policy when a new feature is launched or when new operations become available\. Services don’t remove permissions from an AWS managed policy, so policy updates won’t break your existing permissions\.

Additionally, AWS supports managed policies for job functions that span multiple services\. For example, the **ReadOnlyAccess** AWS managed policy provides read\-only access to all AWS services and resources\. When a service launches a new feature, AWS adds read\-only permissions for new operations and resources\. For a list of job function policies, see [AWS managed policies for job functions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_job-functions.html) in the * IAM User Guide*\.

## AWS managed policy: ROSAManageSubscription<a name="security-iam-awsmanpol-rosafullaccess"></a>

You can attach the `ROSAManageSubscription` policy to your IAM identities\.

This policy grants the required permissions that allow an IAM principal to manage the ROSA subscription\.

 **Permissions details** 

This policy includes the following permissions\.
+  `aws-marketplace:Subscribe` – Grants permission to subscribe to the AWS Marketplace product for ROSA\.
+  `aws-marketplace:ViewSubscriptions` – Allows principals to view subscriptions from AWS Marketplace\. This is required so that the IAM principal can view the available AWS Marketplace subscriptions\.
+  `aws-marketplace:Unsubscribe` – Allows principals to remove subscriptions to AWS Marketplace products\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "aws-marketplace:Subscribe",
                "aws-marketplace:Unsubscribe"
            ],
            "Resource": "*",
            "Condition": {
                "ForAnyValue:StringEquals": {
                    "aws-marketplace:ProductId": [
                        "34850061-abaf-402d-92df-94325c9e947f"
                    ]
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "aws-marketplace:ViewSubscriptions"
            ],
            "Resource": "*"
        }
    ]
}
```

## ROSA updates to AWS managed policies<a name="security-iam-awsmanpol-updates"></a>

View details about updates to AWS managed policies for ROSA since the service began tracking these changes\. For automatic alerts about changes to this page, subscribe to the RSS feed on the [ROSA document history](doc-history.md) page\.


| Change | Description | Date | 
| --- | --- | --- | 
|   Red Hat OpenShift Service on AWS started tracking changes  |   Red Hat OpenShift Service on AWS started tracking changes for its AWS managed policies\.  |  March 2, 2022  | 
# Troubleshooting<a name="troubleshooting-rosa"></a>

This topic covers how to troubleshoot problems that you might encounter when you create and deploy ROSA clusters\.

## Troubleshooting permissions<a name="troubleshoot-iam-permissions"></a>

When you use ROSA, you receive support from both Red Hat and AWS\. Before you can receive support from Red Hat, specifically Red Hat site reliability engineers \(SRE\), you must meet several requirements\. For information about these requirements, see the Red Hat OpenShift [minimum required service control policy documentation](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa_getting_started_iam/rosa-aws-prereqs.html#rosa-minimum-scp_prerequisites)\.

Run the following command to verify that your AWS account has the correct permissions\.

```
rosa verify permissions
```

If you encounter an error, double check if it’s because a [service control policy](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html#orgs_manage_policies_scp) \(SCP\) isn’t applied to your AWS account\. If you’re required to use an SCP, see the [Red Hat Requirements for Customer Cloud Subscriptions](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa_getting_started_iam/rosa-aws-prereqs.html#rosa-minimum-scp_prerequisites) policies page for more information about the minimum required SCP\.

## Troubleshooting ROSA enablement<a name="troubleshooting-rosa-enablement"></a>

ROSA uses AWS Marketplace to facilitate subscription management, billing, and metering\. When you enable this service, the Red Hat console subscribes to AWS Marketplace\. To enable this service, your IAM principal requires `aws-marketplace` subscription permissions\. AWS provides a managed IAM policy that includes the minimum permissions that are necessary to enable ROSA in the AWS Management Console and to manage the ROSA AWS Marketplace subscription\. For more information, see [AWS managed policy: ROSAManageSubscription](security-iam-awsmanpol.md)\.
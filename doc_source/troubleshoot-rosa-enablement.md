# Troubleshoot ROSA enablement errors in the ROSA console<a name="troubleshoot-rosa-enablement"></a>

 ROSA uses AWS Marketplace to facilitate subscription management, billing, and metering\. When you enable ROSA, the AWS ROSA console page subscribes to a ROSA listing on AWS Marketplace\. If your IAM principal is missing required `aws-marketplace` subscription permissions when you enable ROSA in the ROSA console, the console generates an error message\.

This section covers how to troubleshoot AWS Marketplace subscription permission issues that you might encounter when you choose **Enable ROSA ** in the ROSA console\.

With ROSA, you can also receive troubleshooting support from AWS Support and the Red Hat support teams\. For more information, see [Support for ROSA](troubleshooting-rosa.md#rosa-support)\.

**Topics**
+ [AWS Organizations service control policy \(SCP\) is denying required AWS Marketplace permissions](#error-aws-orgs-scp-denies-permissions)
+ [User or role does not have the required AWS Marketplace permissions](#error-iam-lacks-permissions)
+ [Required AWS Marketplace permissions blocked by an administrator](#error-admin-blocked-iam-permissions)

## AWS Organizations service control policy \(SCP\) is denying required AWS Marketplace permissions<a name="error-aws-orgs-scp-denies-permissions"></a>

### Description<a name="_description"></a>

If your AWS Organizations service control policy \(SCP\) isn’t configured to allow the required `aws-marketplace:Subscribe` permission when you choose **Enable ROSA **, the ROSA console generates the following error message: `An error occurred while enabling Red Hat OpenShift Service on AWS (ROSA), because an AWS Organizations Service Control Policy (SCP) is denying required permissions. Contact your AWS Organizations management account administrator, and consult the documentation for troubleshooting.` 

### Solution<a name="_solution"></a>

Your organization’s management account administrator can enable ROSA in the organization’s management account\. Then, they can use AWS License Manager to grant and activate ROSA subscriptions for other accounts within the organization\.

Contact your account administrator and request that they take the following actions to grant ROSA entitlement and activate your license\.

#### Grant entitlement through AWS License Manager<a name="_grant_entitlement_through"></a>

1. Log in to your organization’s management account\.

1. Navigate to the [ROSA console](https://console.aws.amazon.com/rosa)\.

1. Choose **Get started**\.

1. On the **Verify ROSA prerequisites** page, select **I agree to share my contact information with Red Hat**\.

1. Choose **Enable ROSA **\.

1. In the **Enable ROSA across your AWS organization** dialog, choose **Grant entitlements**\. This action creates the service\-linked roles that are needed for the account to share subscriptions with other accounts, and launches the AWS License Manager console\.

1. From the [AWS License Manager console](https://console.aws.amazon.com/license-manager), select the ROSA granted license to view the product details page\.

1. Under **Grants**, choose **Create grant**\.

1. To grant the ROSA license to an individual AWS account in your organization, enter a grant name and the account ID\. If you want to grant the license to all organization accounts at once, enter your organization ID or ARN\.

1. Select the **Auto acceptance** checkbox and choose **Create grant**\.  
![\[Sample workflow for granting<shared id="ROSA"/> licenses in <shared id="LIClong"/>.\]](http://docs.aws.amazon.com/ROSA/latest/userguide/images/rosa-license-manager-create-grant.jpeg)

1. If the **Link AWS Organizations accounts** dialog box appears, choose **Link** to enable auto\-acceptance for managed entitlement distributions\.

1. Choose **Create grant**\.

#### Activate account licenses<a name="_activate_account_licenses"></a>

Before individuals can start using the distributed ROSA license, the license needs to be activated\. After this is done, application owners can deploy ROSA clusters using the ROSA CLI\.

In License Manager, licenses can be activated in bulk by the organization administrator\. Or, licenses can be activated individually by either the administrator or an individual user\. Follow these steps to bulk activate licenses on multiple accounts at once\.

1. Go to your organization’s parent grant page in License Manager\.

1. On the left menu, choose **Granted licenses**\.

1. Choose the ROSA granted license\.

1. Under **Grants**, choose the ROSA grant\.

1. Choose **Activate**\.

1. In the **Activate grant** dialog box, enter *activate* and choose **Activate**\.

After the grant is activated, ROSA is enabled and can be used among all organization users who received the ROSA entitlement\.

## User or role does not have the required AWS Marketplace permissions<a name="error-iam-lacks-permissions"></a>

### Description<a name="_description_2"></a>

If your IAM principal doesn’t have the required `aws-marketplace:Subscribe` permission when you choose **Enable ROSA **, the ROSA console generates the following error message: `An error occurred while enabling Red Hat OpenShift Service on AWS (ROSA), because your user or role does not have the required permissions. ROSAManageSubscription, an AWS managed policy, includes the permissions required to enable ROSA. Consult the documentation and try again.` 

### Solution<a name="_solution_2"></a>

1. Navigate to the [AWS Identity and Access Management \(IAM\) console](https://console.aws.amazon.com/iam/) and attach the AWS managed policy `ROSAManageSubscription` to your IAM identity\. For more information about permissions that are included in the managed policy, see [AWS managed policy: ROSAManageSubscription](security-iam-awsmanpol.md)\.

1. Navigate to the [ROSA console](https://console.aws.amazon.com/rosa)\.

1. Choose **Get started**\.

1. On the **Verify ROSA prerequisites** page, select **I agree to share my contact information with Red Hat**\.

1. Choose **Enable ROSA **\.

If you don’t have permission to view or to update your permission set in IAM, contact your AWS account administrator and ask them to enable ROSA for your account\.

## Required AWS Marketplace permissions blocked by an administrator<a name="error-admin-blocked-iam-permissions"></a>

### Description<a name="_description_3"></a>

If your account administrator blocked the required `aws-marketplace:Subscribe` permission, the ROSA console generates the following error message when you choose **Enable ROSA **: `An error occurred while enabling Red Hat OpenShift Service on AWS (ROSA), because required permissions have been blocked by an administrator. ROSAManageSubscription, an AWS managed policy, includes the permissions required to enable ROSA. Consult the documentation and try again.` 

### Solution<a name="_solution_3"></a>

Contact your AWS account administrator and ask them to take the following action:

1. Navigate to the [ROSA console](https://console.aws.amazon.com/rosa)\.

1. Choose **Get started**\.

1. On the **Verify ROSA prerequisites** page, select **I agree to share my contact information with Red Hat**\.

1. Choose **Enable ROSA **\. This action enables ROSA for all IAM identities under the AWS account\.
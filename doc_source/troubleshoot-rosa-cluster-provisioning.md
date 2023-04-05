# Troubleshoot ROSA cluster provisioning issues<a name="troubleshoot-rosa-cluster-provisioning"></a>

This section contains solutions to issues you may have when provisioning ROSA clusters\.

With ROSA, you can also receive troubleshooting support from AWS Support and the Red Hat support teams\. For more information, see [Support for ROSA](troubleshooting-rosa.md#rosa-support)\.

**Topics**
+ [Access ROSA cluster debug logs](#access-rosa-debug-logs)
+ [Elastic Load Balancing \(ELB\) role does not exist](#elb-role-missing-error)
+ [ROSA cluster fails AWS service quota check during cluster creation](#service-quotas-missing-error)
+ [Troubleshoot ROSA CLI expired offline access tokens](#rosa-cli-expired-token)

## Access ROSA cluster debug logs<a name="access-rosa-debug-logs"></a>

To begin to troubleshoot issues with your application, first review the debug logs\. The ROSA CLI debug logs provide details on the error messages that are produced when a cluster fails to create\.

To display cluster debug information, run the following ROSA CLI command\. In the command, replace `<cluster_name>` with the name of your cluster\.

```
rosa describe cluster -c <cluster_name> --debug
```

## Elastic Load Balancing \(ELB\) role does not exist<a name="elb-role-missing-error"></a>

### Description<a name="_description"></a>

If you didn’t create a load balancer in your AWS account, the `AWSServiceRoleForElasticLoadBalacing` role might have not been created\. If you don’t configure the Elastic Load Balancing role correctly and attempt to create a ROSA cluster, the following error message is returned: `Error creating network Load Balancer: AccessDenied`\.

### Solution<a name="_solution"></a>

1. Check if your account has the `AWSServiceRoleForElasticLoadBalancing` role\.

   ```
   aws iam get-role --role-name "AWSServiceRoleForElasticLoadBalancing"
   ```

1. If you don’t have this role, create the role by running the following command\.

   ```
   aws iam get-role --role-name "AWSServiceRoleForElasticLoadBalancing" || aws iam create-service-linked-role --aws-service-name "elasticloadbalancing.amazonaws.com"
   ```

## ROSA cluster fails AWS service quota check during cluster creation<a name="service-quotas-missing-error"></a>

### Description<a name="_description_2"></a>

To use ROSA, the service quotas for your account may need increased\. For more information, see [ROSA service quotas](service-quotas-rosa.md)\.

### Solution<a name="_solution_2"></a>

1. Run the following command to identify your account’s quotas\.

   ```
   rosa verify quota
   ```
**Note**  
Quotas are different in different AWS Regions\. Make sure to verify each of the quotas for your Regions\.

1. If you need to increase your quota, navigate to the [Service Quotas console](https://console.aws.amazon.com/servicequotas)\.

1. On the navigation pane, choose ** AWS services**\.

1. Choose the service that needs a quota increase\.

1. Select the quota that needs to be increased and choose **Request quota increase**\.

1. For **Request quota increase**, enter the total amount that you want the quota to be and choose **Request**\.

## Troubleshoot ROSA CLI expired offline access tokens<a name="rosa-cli-expired-token"></a>

### Description<a name="_description_3"></a>

If you use the ROSA CLI and your [api\.openshift\.com](https://api.openshift.com/) offline access token expires, an error message appears\. This happens when [sso\.redhat\.com](https://sso.redhat.com) invalidates the token\.

### Solution<a name="_solution_3"></a>

1. Navigate to the [OpenShift Cluster Manager API Token page](https://console.redhat.com/openshift/token/rosa) and choose **Load Token**\.

1. Copy and paste the following authentication command in the terminal\.

   ```
   rosa login --token="<api_token>"
   ```
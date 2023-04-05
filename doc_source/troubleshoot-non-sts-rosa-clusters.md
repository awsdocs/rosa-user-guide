# Troubleshoot non\-STS ROSA cluster issues<a name="troubleshoot-non-sts-rosa-clusters"></a>

This section covers how to troubleshoot issues that you might encounter when provisioning non\-STS ROSA clusters\.

We recommend that you provision ROSA clusters using AWS Security Token Service \(STS\) short\-lived credentials for better security protection\. For more information about provisioning ROSA STS clusters, see [Getting started with ROSA using AWS STS in auto mode](getting-started-sts-auto.md)\.

With ROSA, you can also receive troubleshooting support from AWS Support or the Red Hat support teams\. For more information, see [Support for ROSA](troubleshooting-rosa.md#rosa-support)\.

## Failed to create a cluster with an osdCcsAdmin error<a name="osdccsadmin-error"></a>

**Note**  
This error occurs only when you use the non\-STS method of provisioning ROSA clusters\. To avoid this issue, provision your ROSA clusters using AWS STS\. For more information, see [Getting started with ROSA using AWS STS in auto mode](getting-started-sts-auto.md)\.

### Description<a name="_description"></a>

If your cluster fails to create, you might receive the following error message:

```
Failed to create cluster: Unable to create cluster spec: Failed to get access keys for user 'osdCcsAdmin': NoSuchEntity: The user with name osdCcsAdmin cannot be found.
```

### Solution<a name="_solution"></a>

1. Delete the stack\.

   ```
   rosa init --delete-stack
   ```

1. Reinitialize your account\.

   ```
   rosa init
   ```
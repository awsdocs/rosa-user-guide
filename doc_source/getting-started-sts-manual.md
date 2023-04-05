# Getting started with ROSA using AWS STS in manual mode<a name="getting-started-sts-manual"></a>

The following sections describe how to get started with ROSA using AWS Security Token Service \(AWS STS\) and the ROSA CLI\.

The ROSA CLI uses `auto` mode or `manual` mode to create the IAM resources that are required to provision a ROSA cluster\. `auto` mode immediately creates the required IAM roles and policies and an OpenID Connect \(OIDC\) provider\. `manual` mode outputs the AWS CLI commands that are needed to create the IAM resources\. By using `manual` mode, you can review the generated AWS CLI commands before running them manually\. You can also use `manual` to pass the commands to another administrator or group in your organization so they can create the resources\.

The procedures in this document use `manual` mode to create the required IAM resources\. For instructions on how to deploy a ROSA cluster using `auto` mode, see [Create a ROSA cluster with AWS STS using the default `auto` mode](getting-started-sts-auto.md#getting-started-sts-auto-step-3)\.

**Topics**
+ [Prerequisites](#getting-started-sts-manual-prereqs)
+ [Step 1: Verify ROSA prerequisites](#getting-started-sts-manual-step-1)
+ [Step 2: Create a ROSA cluster with AWS STS using `manual` mode](#getting-started-sts-manual-step-2)
+ [Step 3: Create a cluster administrator for quick cluster access](#getting-started-sts-manual-step-3)
+ [Step 4: Configure an identify provider and grant cluster access](#getting-started-sts-manual-step-4)
+ [Step 5: Grant user access to a cluster](#getting-started-sts-manual-step-5)
+ [Step 6: Grant administrator permissions to a user](#getting-started-sts-manual-step-6)
+ [Step 7: Access a cluster through the web console](#getting-started-sts-manual-step-7)
+ [Step 8: Deploy an application from the Developer Catalog](#getting-started-sts-manual-step-8)
+ [Step 9: Revoke administrator permissions and user access](#getting-started-sts-manual-step-9)
+ [Step 10: Delete a ROSA cluster and AWS STS resources](#getting-started-sts-manual-step-10)

## Prerequisites<a name="getting-started-sts-manual-prereqs"></a>

Before getting started, make sure you completed these actions:
+ Install and configure the latest AWS CLI\. For more information, see [Installing or updating the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)\.
+ Install and configure the latest ROSA CLI and OpenShift Container Platform CLI\. For more information, see [Getting started with the ROSA CLI](https://docs.openshift.com/rosa/rosa_cli/rosa-get-started-cli.html#rosa-setting-up-cli_rosa-getting-started-cli)\.
+  Service Quotas must have the required service quotas set for Amazon EC2, Amazon VPC, Amazon EBS, and Elastic Load Balancing that are needed to create and run a ROSA cluster\. AWS or Red Hat may request service quota increases on your behalf as required for issue resolution\. To view the required quotas, see [ROSA service quotas](service-quotas-rosa.md)\. For more general information about Service Quotas, see [AWS service quotas](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html) in the * AWS General Reference*\.
+ To receive AWS support for ROSA, you must enable AWS Business, Enterprise On\-Ramp, or Enterprise support plans\. Red Hat may request AWS support on your behalf as required for issue resolution\. For more information, see [Support for ROSA](troubleshooting-rosa.md#rosa-support)\. To enable AWS Support, see the [AWS Support page](http://aws.amazon.com/premiumsupport/)\.
+ If you’re using AWS Organizations to manage the AWS accounts that host the ROSA service, the organization’s service control policy \(SCP\) must be configured to allow Red Hat to perform policy actions that’s listed in the SCP without restriction\. For more information, see the [ROSA SCP troubleshooting documentation](troubleshoot-rosa-enablement.md#error-aws-orgs-scp-denies-permissions)\. For more information about SCPs, see [Service control policies \(SCPs\)](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)\.
+ If deploying a ROSA cluster with AWS STS into an enabled AWS Region that’s disabled by default, you must update the security token to version 2 for all the Regions in the AWS account with the following command\.

  ```
  aws iam set-security-token-service-preferences --global-endpoint-token-version v2Token
  ```

  For more information about enabling Regions, see [Managing AWS Regions](https://docs.aws.amazon.com/general/latest/gr/rande-manage.html) in the *AWS General Reference*\.

## Step 1: Verify ROSA prerequisites<a name="getting-started-sts-manual-step-1"></a>

To create a ROSA cluster, your AWS account must meet the prerequisites to use ROSA\. The AWS ROSA console verifies if your account has the necessary AWS Marketplace permissions, service quotas, and the Elastic Load Balancing \(ELB\) service\-linked role named `AWSServiceRoleForElasticLoadBalancing`\. If any of these prerequisites are missing, the ROSA console page provides guidance on how to configure your account to meet the prerequisites\.

1. Navigate to the [ROSA console](https://console.aws.amazon.com/rosa)\.

1. Choose **Get started**\.

1. On the **Verify ROSA prerequisites** page, select **I agree to share my contact information with Red Hat**\.

1. Choose **Enable ROSA **\.

1. Once the page has verified your service quotas meet ROSA prerequisites and the ELB service\-linked role is created, open a new terminal session to create your first ROSA cluster using the ROSA CLI\.

## Step 2: Create a ROSA cluster with AWS STS using `manual` mode<a name="getting-started-sts-manual-step-2"></a>

You can create a ROSA cluster using AWS Security Token Service \(AWS STS\)\. To create a custom cluster, use the `manual` method that’s provided in the ROSA CLI\.

When you create a cluster, you can run `rosa create cluster --interactive` to customize your deployment with a series of interactive prompts\. For more information, see [Interactive cluster creation mode reference](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa-sts-interactive-mode-reference.html) in the Red Hat OpenShift documentation\.

After the cluster is provisioned, a single command is provided in the output\. Run this command to deploy further clusters that use the exact same custom configuration\.

**Note**  
 [AWS shared VPCs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html) aren’t currently supported for ROSA installations\.

1. Create the required account\-wide roles and policies, including the operator policies\. Do so by running the following command\.

   ```
   rosa create account-roles --mode manual
   ```

1. Run the AWS CLI commands generated in the output to create the roles and policies\.

1. Create a cluster with AWS STS in `--interactive` mode to specify any custom settings\.

   ```
   rosa create cluster --interactive --sts
   ```
**Important**  
After you enable etcd encryption for the key values in etcd, you incur a performance overhead of approximately 20%\. The overhead is a result of introducing this second layer of encryption, in addition to the default control plane storage encryption that encrypts the etcd volumes\. Red Hat recommends that you enable etcd encryption only if you specifically require it\.

1. To create the cluster\-specific operator IAM roles, generate the operator policy JSON files in the current working directory and output the AWS CLI commands for review\. Do so by running the following command\.

   ```
   rosa create operator-roles --mode manual --cluster <cluster_name|cluster_id>
   ```

1. Run the AWS CLI commands from the output\.

1. Create the OpenID Connect \(OIDC\) provider the cluster operators use to authenticate\.

   ```
   rosa create oidc-provider --mode auto --cluster <cluster_name|cluster_id>
   ```

1. Check the status of your cluster\.

   ```
   rosa describe cluster --cluster <cluster_name|cluster_id>
   ```
**Note**  
If the provisioning process fails or the `State` field doesn’t change to a ready status after 40 minutes, see [Troubleshoot ROSA cluster provisioning issues](troubleshoot-rosa-cluster-provisioning.md)\.  
To contact AWS Support or Red Hat support for assistance, see [Support for ROSA](troubleshooting-rosa.md#rosa-support)\.

1. Track the progress of the cluster being created by watching the OpenShift installer logs\. You can do so by running the following command\.

   ```
   rosa logs install --cluster <cluster_name|cluster_id> --watch
   ```

## Step 3: Create a cluster administrator for quick cluster access<a name="getting-started-sts-manual-step-3"></a>

Before configuring an identity provider, you can create a user with `cluster-admin` permissions for immediate access to your ROSA cluster\.

1. Create a cluster administrator user\.

   ```
   rosa create admin --cluster=<cluster_name>
   ```
**Note**  
The account may take 1\-2 minutes to become active\.

1. Securely store the generated password in the command output\. If you lose this password, delete and recreate the `cluster-admin` user to regain access\.

   ```
   rosa delete admin --cluster=<cluster_name>
   rosa create admin --cluster=<cluster_name>
   ```

1. Log in to the cluster through the CLI by running the following command\. Replace `<api_url>` and `<cluster_admin_password>` with the credentials for your environment\.

   ```
   oc login <api_url> --username cluster-admin --password <cluster_admin_password>
   ```

1. Verify that you’re logged in as the `cluster-admin` user\.

   ```
   oc whoami
   ```

## Step 4: Configure an identify provider and grant cluster access<a name="getting-started-sts-manual-step-4"></a>

 ROSA includes a built\-in OAuth server\. After your ROSA cluster is created, you must configure OAuth to use an identity provider\. You can then add users to your configured identity provider to grant them access to your cluster\. You can grant these users `cluster-admin` or `dedicated-admin` permissions as required\.

You can configure different identify provider types for your ROSA cluster\. Supported types include GitHub, GitHub Enterprise, GitLab, Google, LDAP, OpenID Connect, and HTPasswd identify providers\.

**Important**  
The HTPasswd identify provider is included only to enable a single, static administrator user to be created\. HTPasswd isn’t supported as a general\-use identity provider for ROSA\.

The following procedure configures a GitHub identity provider as an example\. For instructions on how to configure each of the supported identity provider types, see [Configuring identity providers for AWS STS](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa-sts-config-identity-providers.html#rosa-sts-config-identity-providers)\.

1. Navigate to [github\.com](https://github.com/) and log in to your GitHub account\.

1. If you don’t have a GitHub organization to use for identity provisioning for your ROSA cluster, create one\. For more information, see [the steps in the GitHub documentation](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/creating-a-new-organization-from-scratch)\.

1. Using the ROSA CLI’s interactive mode, configure an identity provider for your cluster\.

   ```
   rosa create idp --cluster=<cluster_name> --interactive
   ```

1. Follow the configuration prompts in the output to restrict cluster access to members of your GitHub organization\.

   ```
   I: Interactive mode enabled.
   Any optional fields can be left empty and a default will be selected.
   ? Type of identity provider: github
   ? Identity provider name: github-1
   ? Restrict to members of: organizations
   ? GitHub organizations: <github_org_name>
   ? To use GitHub as an identity provider, you must first register the application:
     - Open the following URL:
       https://github.com/organizations/<github_org_name>/settings/applications/new?oauth_application%5Bcallback_url%5D=https%3A%2F%2Foauth-openshift.apps.<cluster_name>/<random_string>.p1.openshiftapps.com%2Foauth2callback%2Fgithub-1&oauth_application%5Bname%5D=<cluster_name>&oauth_application%5Burl%5D=https%3A%2F%2Fconsole-openshift-console.apps.<cluster_name>/<random_string>.p1.openshiftapps.com
     - Click on 'Register application'
   ...
   ```

1. Open the URL in the output with the following command\. Replace `<github_org_name>` with the name of your GitHub organization\.

1. On the GitHub web page, choose **Register application** to register a new OAuth application in your GitHub organization\.

1. Use the information from the GitHub OAuth page to populate the remaining `rosa create idp` interactive prompts using the following command\. Replace `<github_client_id>` and `<github_client_secret>` with the credentials from your GitHub OAuth application\.

   ```
   ...
   ? Client ID: <github_client_id>
   ? Client Secret: [? for help] <github_client_secret>
   ? GitHub Enterprise Hostname (optional):
   ? Mapping method: claim
   I: Configuring IDP for cluster '<cluster_name>'
   I: Identity Provider 'github-1' has been created.
      It will take up to 1 minute for this configuration to be enabled.
      To add cluster administrators, see 'rosa grant user --help'.
      To login into the console, open https://console-openshift-console.apps.<cluster_name>.<random_string>.p1.openshiftapps.com and click on github-1.
   ```
**Note**  
It might take about two minutes for the identity provider configuration to become active\. If you configured a `cluster-admin` user, you can run the `oc get pods -n openshift-authentication --watch` command to watch the OAuth pods redeploy with the updated configuration\.

1. Verify the identity provider has been configured correctly using the following command\.

   ```
   rosa list idps --cluster=<cluster_name>
   ```

## Step 5: Grant user access to a cluster<a name="getting-started-sts-manual-step-5"></a>

You can grant a user access to your ROSA cluster by adding them to the configured identity provider\.

The following procedure adds a user to a GitHub organization that’s configured for identity provisioning to the cluster\.

1. Navigate to [github\.com](https://github.com/) and log in to your GitHub account\.

1. Invite users that require ROSA cluster access to your GitHub organization\. For more information, see [Inviting users to join your organization](https://docs.github.com/en/organizations/managing-membership-in-your-organization/inviting-users-to-join-your-organization) in the documentation on Github\.

## Step 6: Grant administrator permissions to a user<a name="getting-started-sts-manual-step-6"></a>

After you add a user to your configured identity provider, you can grant the user `cluster-admin` or `dedicated-admin` permissions for your ROSA cluster\.

### Configure `cluster-admin` permissions<a name="_configure_cluster_admin_permissions"></a>

1. Grant the `cluster-admin` permissions using the following command\. Replace `<idp_user_name>` and `<cluster_name>` with your user and cluster name\.

   ```
   rosa grant user cluster-admin --user=<idp_user_name> --cluster=<cluster_name>
   ```

1. Verify that the user is listed as a member of the `cluster-admins` group\.

   ```
   rosa list users --cluster=<cluster_name>
   ```

### Configure `dedicated-admin` permissions<a name="_configure_dedicated_admin_permissions"></a>

1. Grant the `dedicated-admin` permissions using the following command\. Replace `<idp_user_name>` and `<cluster_name>` with your user and cluster name\.

   ```
   rosa grant user dedicated-admin --user=<idp_user_name> --cluster=<cluster_name>
   ```

1. Verify that the user is listed as a member of the `cluster-admins` group\.

   ```
   rosa list users --cluster=<cluster_name>
   ```

## Step 7: Access a cluster through the web console<a name="getting-started-sts-manual-step-7"></a>

After you create a cluster administrator user or add a user to your configured identity provider, you can log in to your ROSA cluster through the web console\.

1. Obtain the console URL for your cluster by using the following command\. Replace `<cluster_name>` with the name of your cluster\.

   ```
   rosa describe cluster -c <cluster_name> | grep Console
   ```

1. Navigate to the console URL in the output and log in\.
   + If you create a `cluster-admin` user, log in using the provided credentials\.
   + If you configure an identity provider for your cluster, choose the identity provider name in the **Log in with…​** dialog and complete any authorization requests that are presented by your provider\.

## Step 8: Deploy an application from the Developer Catalog<a name="getting-started-sts-manual-step-8"></a>

From the OpenShift Cluster Manager \(OCM\), you can deploy a Developer Catalog test application and expose it with a route\.

1. Navigate to [OpenShift Cluster Manager](https://console.redhat.com/openshift) and choose the cluster you want to deploy the app into\.

1. On the cluster’s page, choose **Open console**\.

1. In the **Administrator** perspective, choose **Home** > **Projects** > **Create Project**\.

1. Enter a name for your project and optionally add a **Display Name** and **Description**\.

1. Choose **Create** to create the project\.

1. Switch to the **Developer** perspective and choose **\+Add**\. Make sure that the selected project is the one that was just created\.

1. In the **Developer Catalog** dialog, choose **All services**\.

1. In the **Developer Catalog** page, choose **Languages** > **JavaScript** from the menu\.

1. Choose **Node\.js**, and then choose **Create Application** to open the **Create Source\-to\-Image Application** page\.
**Note**  
You might need to choose **Clear All Filters** to display the **Node\.js** option\.

1. In the **Git** section, choose **Try Sample**\.

1. In the **Name** field, add a unique name\.

1. Choose **Create**\.
**Note**  
The new application takes several minutes to deploy\.

1. When the deployment is complete, choose the route URL for the application\.

   A new tab in the browser opens with a message similar to the following\.

   ```
   Welcome to your Node.js application on OpenShift
   ```

1. \(Optional\) Delete the application and clean up resources\.

   1. In the **Administrator** perspective, choose **Home** > **Projects**\.

   1. Open the action menu for your project and choose **Delete Project**\.

## Step 9: Revoke administrator permissions and user access<a name="getting-started-sts-manual-step-9"></a>

You can revoke `cluster-admin` or `dedicated-admin` permissions from a user by using the ROSA CLI\.

To revoke access from a user, you must remove the user from your configured identity provider\.

### Revoke `cluster-admin` permissions from a user<a name="_revoke_cluster_admin_permissions_from_a_user"></a>

1. Revoke the `cluster-admin` permission by using the following command\. Replace `<idp_user_name>` and `<cluster_name>` with your user and cluster name\.

   ```
   rosa revoke user cluster-admin --user=<idp_user_name> --cluster=<cluster_name>
   ```

1. Verify that the user isn’t listed as a member of the `cluster-admins` group\.

   ```
   rosa list users --cluster=<cluster_name>
   ```

### Revoke `dedicated-admin` permissions from a user<a name="_revoke_dedicated_admin_permissions_from_a_user"></a>

1. Revoke the `dedicated-admin` permission using the following command\. Replace `<idp_user_name>` and `<cluster_name>` with your user and cluster name\.

   ```
   rosa revoke user dedicated-admin --user=<idp_user_name> --cluster=<cluster_name>
   ```

1. Verify that the user isn’t listed as a member of the `dedicated-admins` group\.

   ```
   rosa list users --cluster=<cluster_name>
   ```

### Revoke user access to a cluster<a name="_revoke_user_access_to_a"></a>

You can revoke cluster access for an identify provider user by removing them from the configured identity provider\.

You can configure different types of identity providers for your ROSA cluster\. The following procedure revokes cluster access for a member of a GitHub organization\.

1. Navigate to [github\.com](https://github.com/) and log in to your GitHub account\.

1. Remove the user from your GitHub organization\. For more information, see [Removing a member from your organization](https://docs.github.com/en/organizations/managing-membership-in-your-organization/removing-a-member-from-your-organization) in the GitHub documentation\.

## Step 10: Delete a ROSA cluster and AWS STS resources<a name="getting-started-sts-manual-step-10"></a>

You can use the ROSA CLI to delete a ROSA cluster that uses the AWS Security Token Service \(AWS STS\)\. You can also use the ROSA CLI to delete the AWS Identity and Access Management \(IAM\) account\-wide roles, the cluster\-specific operator roles, and the OpenID Connect \(OIDC\) provider\. To delete the account\-wide inline and operator policies, you can use the IAM console\.

**Important**  
Account\-wide IAM roles and policies might be used by other ROSA clusters in the same account\. You must only remove the resources if they aren’t required by other clusters\.

1. Delete the ROSA cluster and watch the logs using the following command\. Replace `<cluster_name>` with the name or ID of your cluster\.

   ```
   rosa delete cluster --cluster=<cluster_name> --watch
   ```
**Important**  
You must wait for cluster to completely delete before you remove the IAM roles, policies, and OIDC provider\. The account\-wide roles are required to delete the resources created by the installer\. The cluster\-specific operator roles are required to clean up the resources created by the OpenShift operators\. The operators use the OIDC provider to authenticate\.

1. Delete the OIDC provider that the cluster operators use to authenticate\.

   ```
   rosa delete oidc-provider -c <cluster_id> --mode auto
   ```

1. Delete the cluster\-specific operator IAM roles\.

   ```
   rosa delete operator-roles -c <cluster_id> --mode auto
   ```

1. Delete the account\-wide roles using the following command\. Replace `<prefix>` with the prefix of the account\-wide roles to delete\. If you specified a custom prefix when creating the account\-wide roles, specify the default `ManagedOpenShift` prefix\.

   ```
   rosa delete account-roles --prefix <prefix> --mode auto
   ```

1. Delete the account\-wide inline and operator IAM policies\.

   1. Log in to the [IAM console](https://console.aws.amazon.com/iamv2/home#/home)\.

   1. On the left menu under **Access management**, choose **Policies**\.

   1. Select the account\-wide policy you want to delete and choose **Actions** > **Delete**\.

   1. Enter the policy name and choose **Delete**\.

   1. Repeat this step to delete each of the account\-wide inline and operator policies for the cluster\.
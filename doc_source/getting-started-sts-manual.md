# Getting started with ROSA using AWS STS in manual mode<a name="getting-started-sts-manual"></a>

The following sections describe how to get started with ROSA using AWS Security Token Service \(AWS STS\) and the ROSA CLI\.

The ROSA CLI uses `auto` mode or `manual` mode to create the IAM resources that are required to provision a ROSA cluster\. `auto` mode immediately creates the required IAM roles and policies and an OpenID Connect \(OIDC\) provider\. `manual` mode outputs the AWS CLI commands that are needed to create the \{∏iam\} resources\. By using `manual` mode, you can review the generated AWS CLI commands before running them manually\. You can also use `manual` to pass the commands to another administrator or group in your organization so they can create the resources\.

The procedures in this document use `manual` mode to create the required IAM resources\. For instructions on how to deploy a ROSA cluster using `auto` mode, see [Create a ROSA cluster with AWS STS using the default `auto` mode](getting-started-sts-auto.md#getting-started-sts-auto-step-3)\.

For more information about the `auto` and `manual` deployment modes, see [Understanding the auto and manual deployment modes](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa-sts-creating-a-cluster-with-customizations.html#rosa-understanding-deployment-modes_rosa-sts-creating-a-cluster-with-customizations)\.

## Prerequisites<a name="getting-started-sts-manual-prereqs"></a>

Before getting started, make sure you completed these actions:
+ Install and configure the latest AWS CLI\. For more information, see [Installing or updating the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)\.
+ Install and configure the latest ROSA CLI and OpenShift Container Platform CLI\. For more information, see [Getting started with the rosa CLI](https://docs.openshift.com/rosa/rosa_cli/rosa-get-started-cli.html#rosa-setting-up-cli_rosa-getting-started-cli)\.
+  Service Quotas must have the required service quotas set for Amazon EC2, Amazon VPC, Amazon EBS, and Elastic Load Balancing that are needed to create and run a ROSA cluster\. To view the required quotas, see [Red Hat’s documentation on required AWS service quotas](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa_getting_started_iam/rosa-required-aws-service-quotas.html#rosa-required-aws-service-quotas)\. For more information about Service Quotas, see [Required AWS service quotas](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html)\.
+  AWS requires enabling AWS Business, Enterprise On\-Ramp, or Enterprise support plans for ROSA support\. Red Hat might request AWS Support on your behalf and request AWS resource quota increases as required for your issue resolution\. For more information, see [AWS Support](http://aws.amazon.com/premiumsupport/) and [AWS prerequisites for ROSA](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa_getting_started_iam/rosa-aws-prereqs.html#prerequisites)\.
+ If you’re using AWS Organizations to manage the AWS accounts hosting the ROSA service, your organization’s service control policy \(SCP\) must be configured to allow Red Hat to perform policy actions that are listed in the SCP without restriction\. For more information about AWS and Red Hat infrastructure requirements for ROSA, see [Red Hat’s documentation on the topic](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa_getting_started_iam/rosa-aws-prereqs.html#prerequisites)\. For more information about SCPs, see [Service control policies \(SCPs\)](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)\.
+ If deploying a ROSA cluster with AWS STS into an enabled AWS Region that’s disabled by default, you must update the security token to version 2 for all the Regions in your AWS account\. You can do so with the following command\.

  ```
  aws iam set-security-token-service-preferences --global-endpoint-token-version v2Token
  ```

  For more information about enabling Regions, see [Managing AWS Regions](https://docs.aws.amazon.com/general/latest/gr/rande-manage.html) in *AWS General Reference*\.

## Step 1: Enable ROSA<a name="getting-started-sts-manual-step-1"></a>

ROSA can be enabled in the AWS Management Console\.

1. Navigate to [Red Hat OpenShift on AWS](https://console.aws.amazon.com/rosa/home)\.

1. Choose **Enable ROSA**\.

## Step 2: Create the Elastic Load Balancing role<a name="getting-started-sts-manual-step-2"></a>

To create a ROSA cluster, the `AWSServiceRoleForElasticLoadBalacing` role must be in place\.

**Note**  
If you attempt to create a ROSA cluster without the Elastic Load Balancing role properly configured, the following error occurs: `Error creating network Load Balancer: AccessDenied`\.

1. To check if the `AWSServiceRoleForElasticLoadBalancing` role exists for your account, run the following command\.

   ```
   aws iam get-role --role-name "AWSServiceRoleForElasticLoadBalancing"
   ```

   The following output confirms the role exists\.

   ```
   ROLE    arn:aws:iam::<aws_account_number>:role/aws-service-role/elasticloadbalancing.amazonaws.com/AWSServiceRoleForElasticLoadBalancing  2018-09-27T19:49:23+00:00       Allows ELB to call AWS services on your behalf. 3600      /aws-service-role/elasticloadbalancing.amazonaws.com/   <role_id>   AWSServiceRoleForElasticLoadBalancing
   ASSUMEROLEPOLICYDOCUMENT        2012-10-17
   STATEMENT       sts:AssumeRole  Allow
   PRINCIPAL       elasticloadbalancing.amazonaws.com
   ROLELASTUSED    2022-01-06T09:27:57+00:00       us-east-1
   ```

1. If the role doesn’t exist, create it by running the following command\.

   ```
   aws iam create-service-linked-role --aws-service-name "elasticloadbalancing.amazonaws.com"
   ```

## Step 3: Create a ROSA cluster with AWS STS using `manual` mode<a name="getting-started-sts-manual-step-3"></a>

You can create a ROSA cluster using the AWS Security Token Service \(AWS STS\)\. To create a custom cluster, use the `manual` method that’s provided in the Red Hat OpenShift Cluster Manager \(OCM\) or the ROSA CLI\. This section provides instructions on how to create a custom cluster using the `manual` method for both OCM and the ROSA CLI\.

### Create a custom ROSA cluster using OpenShift Cluster Manager \(OCM\)<a name="_create_a_custom_rosa_using_openshift_cluster_manager_ocm"></a>

When using Red Hat OpenShift Cluster Manager \(OCM\) and AWS STS to create a ROSA cluster, you can customize your installation interactively\.

**Important**  
Only public and AWS PrivateLink clusters are supported with AWS STS\. Regular private clusters \(non\-AWS PrivateLink\) aren’t available for use with AWS STS\.  
For information about deploying private \(non\-AWS PrivateLink\) clusters, see [Configuring a private \{cluster\}](https://docs.openshift.com/rosa/rosa_cluster_admin/cloud_infrastructure_access/rosa-private-cluster.html)\.  
For more information about different ways to get started with ROSA, see [Getting started with ROSA](getting-started.md)\.

**Note**  
 [AWS shared VPCs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html) aren’t currently supported for ROSA installations\.

1. Navigate to [OpenShift Cluster Manager](https://console.redhat.com/openshift) and choose **Create cluster **\.

1. On the **Create an OpenShift cluster ** page, scroll down to the **Red Hat OpenShift on AWS \(ROSA\)** row under **Managed services** and choose **Create cluster **\.

1. Choose the checkbox to acknowledge that you read and completed the prerequisites\.

1. For **Associated AWS account **, choose an AWS account\. If no associated accounts are found, choose **Associate AWS account ** and follow the wizard prompts to associate a new account\.

1. If the required IAM account roles aren’t detected automatically and listed on the **Accounts and roles** page, create the required account\-wide roles and policies in the ROSA CLI and choose **Refresh ARNs**\. Do so using the following command\.

   ```
   rosa create account-roles
   ```

1. Choose **Next**\.

1. On the ** Cluster details** page, provide a name for your cluster and specify the cluster details\.
**Note**  
Select **Enable additional etcd encryption** if you require etcd key value encryption\. With this option, etcd key values are encrypted, but not the keys\.  
With etcd encryption enabled, you incur a performance overhead of approximately 20%\. The overhead is a result of introducing this second layer of encryption, in addition to the default control plane storage encryption for etcd volumes\. Consider enabling etcd encryption only if you require it for your use case\.  
![\[sample ROSA <shared id="cluster"/> configuration on the OCM console Cluster details page\]](http://docs.aws.amazon.com/ROSA/latest/userguide/images/ocm-rosa-cluster-settings.png)

1. Choose **Next**\.

1. On the **Default machine pool** page, choose a compute node instance type\.

1. \(Optional\) Configure auto scaling for the default machine pool\.

   1. Choose **Enable autoscaling** to automatically scale the number of machines in your default machine pool to meet your deployment needs\.

   1. Set the minimum and maximum node count quotas for auto scaling\. The cluster autoscaler doesn’t reduce or increase the default machine pool node count beyond the quotas that you specify\.

1. Choose **Next**\.

1. On the **Networking configuration** page, choose **Public** or **Private** and then choose **Next**\.
**Important**  
If you use private API endpoints, you must use an existing VPC and AWS PrivateLink\. You can’t access your cluster until you update the network settings in your account\.

1. On the **CIDR ranges** page, configure custom classless inter\-domain routing \(CIDR\) ranges or use the defaults that are provided and choose **Next**\.
**Important**  
CIDR configurations can’t be changed later\. Confirm your selections with your network administrator before proceeding\.

1. On the ** Cluster roles and policies** page, choose **Manual** and then choose **Next**\.

1. On the ** Cluster update strategy** page, leave **Individual updates** selected and choose **Next**\.

1. Review your cluster configuration and choose **Create cluster **\.

1. In the installation dialog that opens, select the ** AWS CLI ** or **ROSA CLI** tab and follow the on\-screen instructions to create the operator roles and OIDC provider to complete cluster installation\.
**Note**  
If the provisioning process fails or the `State` field doesn’t change to a ready status after 40 minutes pass, refer to the Red Half Openshift documentation\. For troubleshooting information specifically, see [Troubleshooting installations](https://docs.openshift.com/rosa/rosa_support/rosa-troubleshooting-installations.html)\.  
To contact Red Hat Support for assistance, see [Getting support for Red Hat OpenShift Service on AWS](https://docs.openshift.com/rosa/rosa_architecture/rosa-getting-support.html#rosa-getting-support)\.

### Create a custom ROSA cluster using the ROSA CLI<a name="_create_a_custom_rosa_using_the_rosa_cli"></a>

When you create a ROSA cluster that uses AWS STS, you can customize your installation in an interactive way\.

When you run `rosa create cluster --interactive` when you create a cluster, you’re presented with a series of interactive prompts to customize your deployment\. For more information, see [Interactive cluster creation mode reference](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa-sts-interactive-mode-reference.html) in the Red Hat OpenShift documentation\.

After the cluster is provisioned, a single command is provided in the output\. Run this command to deploy further clusters that use the exact same custom configuration\.

**Important**  
Only public and AWS PrivateLink clusters are supported with AWS STS\. Regular private clusters \(non\-AWS PrivateLink\) aren’t available for use with AWS STS\.  
For information about how to deploy private \(non\-AWS PrivateLink\) clusters, see [Configuring a private cluster](https://docs.openshift.com/rosa/rosa_cluster_admin/cloud_infrastructure_access/rosa-private-cluster.html)\.  
For more information about different ways to get started with ROSA, see [Getting started with ROSA](getting-started.md)\.

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
If the provisioning process fails or the `State` field doesn’t change to a ready status after 40 minutes, check the troubleshooting documentation for details\. For more information, see [Troubleshooting installations](https://docs.openshift.com/rosa/rosa_support/rosa-troubleshooting-installations.html)\.  
To contact Red Hat Support for assistance, see [Getting support for Red Hat OpenShift Service on AWS](https://docs.openshift.com/rosa/rosa_architecture/rosa-getting-support.html#rosa-getting-support)\.

1. Track the progress of the cluster being created by watching the OpenShift installer logs\. You can do so by running the following command\.

   ```
   rosa logs install --cluster <cluster_name|cluster_id> --watch
   ```

## Step 4: Create a cluster administrator for quick cluster access<a name="getting-started-sts-manual-step-4"></a>

Before configuring an identity provider, you can create a user with `cluster-admin` permissions for immediate access to your ROSA cluster\.

1. Create a cluster administrator user\.

   ```
   rosa create admin --cluster=<cluster_name>
   ```
**Note**  
It might take a minute or so for the account to become active\.

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

## Step 5: Configure an identify provider and grant cluster access<a name="getting-started-sts-manual-step-5"></a>

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

## Step 6: Grant user access to a cluster<a name="getting-started-sts-manual-step-6"></a>

You can grant a user access to your ROSA cluster by adding them to the configured identity provider\.

The following procedure adds a user to a GitHub organization that’s configured for identity provisioning to the cluster\.

1. Navigate to [github\.com](https://github.com/) and log in to your GitHub account\.

1. Invite users that require ROSA cluster access to your GitHub organization\. For more information, see [Inviting users to join your organization](https://docs.github.com/en/organizations/managing-membership-in-your-organization/inviting-users-to-join-your-organization) in the documentation on Github\.

## Step 7: Grant administrator permissions to a user<a name="getting-started-sts-manual-step-7"></a>

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

## Step 8: Access a cluster through the web console<a name="getting-started-sts-manual-step-8"></a>

After you create a cluster administrator user or add a user to your configured identity provider, you can log in to your ROSA cluster through the web console\.

1. Obtain the console URL for your cluster by using the following command\. Replace `<cluster_name>` with the name of your cluster\.

   ```
   rosa describe cluster -c <cluster_name> | grep Console
   ```

1. Navigate to the console URL in the output and log in\.
   + If you create a `cluster-admin` user, log in using the provided credentials\.
   + If you configure an identity provider for your cluster, choose the identity provider name in the **Log in with…​** dialog and complete any authorization requests that are presented by your provider\.

## Step 9: Deploy an application from the Developer Catalog<a name="getting-started-sts-manual-step-9"></a>

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

## Step 10: Revoke administrator permissions and user access<a name="getting-started-sts-manual-step-10"></a>

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

## Step 11: Delete a ROSA cluster and AWS STS resources<a name="getting-started-sts-manual-step-11"></a>

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
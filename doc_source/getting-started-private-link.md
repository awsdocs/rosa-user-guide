# Getting started with ROSA using AWS PrivateLink<a name="getting-started-private-link"></a>

ROSA clusters can be deployed in a few different ways: public, private, or private with AWS PrivateLink\. For both public and private cluster configurations, the OpenShift cluster has access to the internet, and privacy is set on the application workloads at the application layer\. If you require both the OpenShift cluster and the application workloads to be private, you can configure an AWS PrivateLink cluster on ROSA\.

With ROSA PrivateLink clusters, Red Hat Site Reliability Engineering \(SRE\) teams no longer require the cluster to be public to monitor or perform actions on your behalf\. Instead, the SRE teams can access the cluster by using a private subnet connected to the cluster’s PrivateLink endpoint\.

For more information about AWS PrivateLink, see [What is AWS PrivateLink?](https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html) 

## Prerequisites<a name="getting-started-private-link-prereqs"></a>

Before getting started, make sure you complete these actions:
+ Install and configure the latest AWS CLI\. For more information, see [Installing or updating the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)\.
+ Install and configure the latest ROSA CLI and OpenShift Container Platform CLI\. For more information, see [Getting started with the rosa CLI](https://docs.openshift.com/rosa/rosa_cli/rosa-get-started-cli.html#rosa-setting-up-cli_rosa-getting-started-cli)\.
+  Service Quotas must have the required service quotas set for Amazon EC2, Amazon VPC, Amazon EBS, and Elastic Load Balancing that are needed to create and run a ROSA cluster\. To view the required quotas, see [Red Hat’s documentation on required AWS service quotas](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa_getting_started_iam/rosa-required-aws-service-quotas.html#rosa-required-aws-service-quotas)\. For more information about Service Quotas, see [Required AWS service quotas](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html)\.
+  AWS requires enabling AWS Business, Enterprise On\-Ramp, or Enterprise support plans for ROSA support\. Red Hat might request AWS Support on your behalf and request AWS resource quota increases as required for your requirements\. For more information, see [AWS Support](http://aws.amazon.com/premiumsupport/) and [AWS prerequisites for ROSA](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa_getting_started_iam/rosa-aws-prereqs.html#prerequisites) in the Red Hat OpenShift documentation\.
+ If you’re using AWS Organizations to manage the AWS accounts that host the ROSA service, your organization’s service control policy \(SCP\) must be configured to allow Red Hat to perform policy actions that are listed in the SCP without restriction\. For more information about AWS and Red Hat infrastructure requirements for ROSA, see [AWS prerequisites for ROSA](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa_getting_started_iam/rosa-aws-prereqs.html#prerequisites) in the Red Hat OpenShift documentation\. For more information about SCPs, see [Service control policies \(SCPs\)](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)\.
+ If deploying a ROSA cluster with AWS STS into an enabled AWS Region that’s disabled by default, you must update the security token to version 2 for all the Regions in the AWS account with the following command\.

  ```
  aws iam set-security-token-service-preferences --global-endpoint-token-version v2Token
  ```

  For more information about enabling Regions, see [Managing AWS Regions](https://docs.aws.amazon.com/general/latest/gr/rande-manage.html) in the *AWS General Reference*\.

## Step 1: Enable ROSA<a name="getting-started-private-link-step-1"></a>

ROSA can be enabled in the AWS Management Console by following these steps\.

1. Navigate to [Red Hat OpenShift on AWS](https://console.aws.amazon.com/rosa/home)\.

1. Choose **Enable ROSA**\.

## Step 2: Create the Elastic Load Balancing role<a name="getting-started-private-link-step-2"></a>

To create a ROSA cluster, the `AWSServiceRoleForElasticLoadBalacing` role must be in place\.

**Note**  
If you didn’t properly configure the Elastic Load Balancing role and attempt to create a ROSA cluster, you receive the following error message: `Error creating network Load Balancer: AccessDenied`\.

1. Check if the `AWSServiceRoleForElasticLoadBalancing` role exists for your account by running the following command\.

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

1. If the role doesn’t exist, create one by running the following command\.

   ```
   aws iam create-service-linked-role --aws-service-name "elasticloadbalancing.amazonaws.com"
   ```

## Step 3: Create Amazon VPC architecture for the AWS PrivateLink use case<a name="getting-started-private-link-step-3"></a>

To create a ROSA private cluster with AWS PrivateLink, you must first configure your own Amazon Virtual Private Cloud \(VPC\) architecture to deploy your solution into\. The following procedure uses the AWS CLI to create both a public and private subnet\. All cluster resources are in the private subnet\. The public subnet routes outbound traffic by using a NAT gateway to the internet\.

This example uses the CIDR block `10.0.0.0/16` for the Amazon VPC\. However, you can choose a different CIDR block\. For more information, see [VPC sizing](https://docs.aws.amazon.com/vpc/latest/userguide/configure-your-vpc.html#vpc-sizing)\.

**Important**  
When creating subnets, make sure that subnets are created to an Availability Zone that has ROSA instance types available\. If you don’t choose a specific Availability Zone, the subnet is created in any one of the Availability Zones in the AWS Region that you specify\. For example, ROSA can’t be installed in `us-east-1e`, but `us-east-1b` works fine\.  
To specify a specific Availability Zone, use the `--availability zone` argument in the `create-subnet` command\. You can use the `rosa list instance-types` command to list all ROSA instance types available\. To check if an instance type is available for a given Availability Zone, use the following command\.  

```
aws ec2 describe-instance-type-offerings --location-type availability-zone --filters Name=location,Values=<availability_zone> --region <region> --output text | egrep "<instance_type>"
```

1. Set an environment variable for the ROSA cluster name by running the following command\.

   ```
    ROSA_CLUSTER_NAME=private-link
   ```

1. Create a VPC with a `10.0.0.0/16` CIDR block\.

   ```
    aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text
   ```

   The preceding command returns the ID of the new VPC\. The following is an example output\.

   ```
   vpc-0410832ee325aafea
   ```

1. Using the VPC ID from the previous step, tag the VPC using the `ROSA_CLUSTER_NAME` variable\.

   ```
   aws ec2 create-tags --resources vpc-0410832ee325aafea --tags Key=Name,Value=$ROSA_CLUSTER_NAME
   ```

1. Enable DNS hostname support on the VPC\.

   ```
   aws ec2 modify-vpc-attribute --vpc-id vpc-0410832ee325aafea --enable-dns-hostnames
   ```

1. Create a subnet in the VPC with a `10.0.1.0/24` CIDR block and a `public` name tag\.

   ```
   aws ec2 create-subnet --vpc-id vpc-0410832ee325aafea --cidr-block 10.0.1.0/24 --availability-zone us-east-1b --query Subnet.SubnetId --output text
   
   aws ec2 create-tags --resources subnet-0b6a7e8cbc8b75920 --tags Key=Name,Value=$ROSA_CLUSTER_NAME-public
   ```

1. Create a subnet in the VPC with a `10.0.0.0/24` CIDR block and a `private` name tag\.

   ```
   aws ec2 create-subnet --vpc-id vpc-0410832ee325aafea --cidr-block 10.0.0.0/24 --availability-zone us-east-1b --query Subnet.SubnetId --output text
   
   aws ec2 create-tags --resources subnet-00964ef8b1a168d36 --tags Key=Name,Value=$ROSA_CLUSTER_NAME-private
   ```

1. Create an internet gateway for outbound traffic and attach it to the VPC\.

   ```
   aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
   
   aws ec2 create-tags --resources igw-09d37a48c404ae6c7 --tags Key=Name,Value=$ROSA_CLUSTER_NAME
   
   aws ec2 attach-internet-gateway --vpc-id vpc-0410832ee325aafea --internet-gateway-id igw-09d37a48c404ae6c7
   ```

1. Create a route table for outbound traffic and associate it to the public subnet\.

   ```
   aws ec2 create-route-table --vpc-id vpc-0410832ee325aafea --query RouteTable.RouteTableId --output text
   
   aws ec2 create-tags --resources rtb-07cf05223b5c91c43 --tags Key=Name,Value=$ROSA_CLUSTER_NAME
   
   aws ec2 create-route --route-table-id rtb-07cf05223b5c91c43 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-09d37a48c404ae6c7
   
   aws ec2 describe-route-tables --route-table-id rtb-07cf05223b5c91c43
   
   aws ec2 associate-route-table --subnet-id subnet-0b6a7e8cbc8b75920 --route-table-id rtb-07cf05223b5c91c43
   ```

1. Create a NAT gateway in the public subnet for outgoing traffic from the private network\.

   ```
   aws ec2 allocate-address --domain vpc --query AllocationId --output text
   
   aws ec2 create-nat-gateway --subnet-id subnet-0b6a7e8cbc8b75920 --allocation-id eipalloc-04e717d29eed16943  --query NatGateway.NatGatewayId --output text
   
   aws ec2 create-tags --resources eipalloc-04e717d29eed16943  --resources nat-08caf8cc0cdabe691 --tags Key=Name,Value=$ROSA_CLUSTER_NAME
   ```

1. Create a route table for the private subnet to the NAT gateway\.

   ```
   aws ec2 create-route-table --vpc-id vpc-0410832ee325aafea --query RouteTable.RouteTableId --output text
   
   aws ec2 create-tags --resources rtb-0614a89d026794f7a eipalloc-04e717d29eed16943 --tags Key=Name,Value=$ROSA_CLUSTER_NAME-private
   
   aws ec2 create-route --route-table-id rtb-0614a89d026794f7a --destination-cidr-block 0.0.0.0/0 --gateway-id nat-08caf8cc0cdabe691
   
   aws ec2 associate-route-table --subnet-id subnet-00964ef8b1a168d36 --route-table-id rtb-0614a89d026794f7a
   ```

## Step 4: Create an AWS PrivateLink cluster<a name="getting-started-private-link-step-4"></a>

With AWS PrivateLink, you can use the ROSA CLI to create a cluster with a single Availability Zone \(Single\-AZ\) or multiple Availability Zones \(Multi\-AZ\)\. In either case, your machine’s CIDR value must match your VPC’s CIDR value\.

The following procedure uses the `rosa create cluster` command to create a Single\-AZ ROSA cluster\. To create a Multi\-AZ cluster, specify `multi-az` in the command and the private subnet IDs for each private subnet you want you to deploy to\.

**Important**  
If you use a firewall, you must configure it so that ROSA can access the sites that it requires to function\.  
For more information, see [AWS PrivateLink firewall prerequisites](https://docs.openshift.com/rosa/rosa_planning/rosa-sts-aws-prereqs.html#osd-aws-privatelink-firewall-prerequisites_rosa-sts-aws-prereqs) in the Red Hat OpenShift documentation\.

1. Create a Single\-AZ ROSA cluster by running the following command\.

   ```
   rosa create cluster --private-link --cluster-name=<cluster-name> --machine-cidr=10.0.0.0/16 --subnet-ids=<private-subnet-id>
   ```
**Note**  
To create an AWS PrivateLink cluster using AWS Security Token Service \(AWS STS\) short\-lived credentials, append `--sts --mode auto` or `sts --mode manual` to the end of the `rosa create cluster` command\. For more information about AWS STS `auto` and `manual` deployments, see [Understanding the auto and manual deployment modes](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa-sts-creating-a-cluster-with-customizations.html#rosa-understanding-deployment-modes_rosa-sts-creating-a-cluster-with-customizations)\.

1. Create the cluster operator IAM roles by following the interactive prompts\.

   ```
   rosa create operator-roles --interactive -c <cluster_name>
   ```

1. Create the OpenID Connect \(OIDC\) provider the cluster operators use to authenticate\.

   ```
   rosa create oidc-provider --interactive -c <cluster_name>
   ```

1. Check the cluster status\.

   ```
   rosa describe cluster --cluster=<cluster_name>
   ```
**Note**  
It might take up to 40 minutes for the cluster `State` field to show the `ready` status\. If provisioning fails or doesn’t show as `ready` after 40 minutes, see [Red Hat Troubleshooting installations documentation](https://docs.openshift.com/rosa/rosa_support/rosa-troubleshooting-installations.html)\.

1. Enter the following command to watch the OpenShift installer logs and track cluster progress\.

   ```
   rosa logs install --cluster=<cluster_name> --watch
   ```

## Step 5: Configure AWS PrivateLink DNS forwarding<a name="getting-started-private-link-step-5"></a>

With AWS PrivateLink clusters, a public hosted zone and a private hosted zone are created in Route 53\. With the private hosted zone, records within the zone are resolvable only from within the VPC that it’s assigned to\.

The Let’s Encrypt DNS\-01 validation requires a public zone so that valid and publicly trusted certificates can be issued for the domain\. The validation records are deleted after Let’s Encrypt validation is complete\. However, the zone is still required for issuing and renewing these certificates, which are typically required every 60 days\. Although these zones usually appear empty, a public zone serves a critical role in the validation process\.

For more information about AWS private hosted zones, see [Working with private zones](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zones-private.html)\. For more information about public hosted zones, see [Working with public hosted zones](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/AboutHZWorkingWith.html)\.

### Configure a Route 53 Resolver inbound endpoint<a name="_configure_a_inbound_endpoint"></a>

To allow for records such as `api.<cluster_domain>` and `*.apps.<cluster_domain>` to resolve outside of the VPC, configure a Route 53 Resolver inbound endpoint\.

1. Open the Route 53 console\.

1. In the navigation pane under **Resolver**, choose **Inbound endpoints**\.

1. Choose **Configure endpoints**\.

1. In the upper right, use the AWS Region selector to choose the Region that contains the VPC used for the AWS PrivateLink cluster\.

1. Under **Basic configuration**, choose **Inbound only** and then choose **Next**\.

1. On the **Configure inbound endpoint** page, complete the **General settings for inbound endpoint** section\. Under **Security group for this endpoint**, choose a security group that allows inbound UDP and TCP traffic from the remote network on destination port 53\.

1. In the **IP address** section, choose the Availability Zones and private subnets that were used when creating the AWS PrivateLink cluster and choose **Next**\.

1. \(Optional\) Complete the **Tags** section\.

1. Choose **Submit**\.

### Configure DNS forwarding for the AWS PrivateLink cluster<a name="_configure_dns_forwarding_for_the_cluster"></a>

After the Route 53 Resolver internal endpoint is associated and operational, configure DNS forwarding so DNS queries can be handled by the designated servers on your network\.

1. Configure your corporate network to forward DNS queries to those IP addresses for the top\-level domain, such as `drow-pl-01.htno.p1.openshiftapps.com`\.

1. If you’re forwarding DNS queries from one VPC to another VPC, follow the instructions in [Managing forwarding rules](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver-rules-managing.html)\.

1. If you’re configuring your remote network DNS server, see your specific DNS server documentation to configure selective DNS forwarding for the installed cluster domain\.

## Step 6: Create a cluster administrator for quick cluster access<a name="getting-started-private-link-step-6"></a>

Before configuring an identity provider, you can create a user with `cluster-admin` permissions for immediate access to your ROSA cluster\.

1. Create a cluster administrator user by running the following command\.

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

1. Log in to the cluster through the CLI using the following command\. Replace `<api_url>` and `<cluster_admin_password>` with the credentials for your environment\.

   ```
   oc login <api_url> --username cluster-admin --password <cluster_admin_password>
   ```

1. Verify you are logged in as the `cluster-admin` user\.

   ```
   oc whoami
   ```

## Step 7: Configure an identify provider and grant cluster access<a name="getting-started-private-link-step-7"></a>

ROSA includes a built\-in OAuth server\. After your ROSA cluster is created, you must configure OAuth to use an identity provider\. You can then add users to your configured identity provider to grant them access to your cluster\. You can grant these users `cluster-admin` or `dedicated-admin` permissions as required\.

You can configure different identify provider types for your ROSA cluster\. The supported types include GitHub, GitHub Enterprise, GitLab, Google, LDAP, OpenID Connect, and HTPasswd identify providers\.

**Important**  
The HTPasswd identify provider is included only to enable a single, static administrator user to be created\. HTPasswd isn’t supported as a general\-use identity provider for ROSA\.

The following procedure configures a GitHub identity provider as an example\. For instructions on how to configure each of the supported identity provider types, see [Configuring identity providers for AWS STS](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa-sts-config-identity-providers.html#rosa-sts-config-identity-providers)\.

1. Navigate to [github\.com](https://github.com/) and log in to your GitHub account\.

1. If you don’t have a GitHub organization to use for identity provisioning for your ROSA cluster, create one\. For more information, see [the steps in the GitHub documentation](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/creating-a-new-organization-from-scratch)\.

1. Using the ROSA CLI’s interactive mode, configure an identity provider for your cluster by running the following command\.

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

1. Open the URL in the output, replacing `<github_org_name>` with the name of your GitHub organization\.

1. On the GitHub web page, choose **Register application** to register a new OAuth application in your GitHub organization\.

1. Use the information from the GitHub OAuth page to populate the remaining `rosa create idp` interactive prompts, replacing `<github_client_id>` and `<github_client_secret>` with the credentials from your GitHub OAuth application\.

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
It might take around two minutes for the identity provider configuration to become active\. If you configured a `cluster-admin` user, you can run the `oc get pods -n openshift-authentication --watch` command to watch the OAuth pods redeploy with the updated configuration\.

1. Verify the identity provider has been configured correctly\.

   ```
   rosa list idps --cluster=<cluster_name>
   ```

## Step 8: Grant user access to a cluster<a name="getting-started-private-link-step-8"></a>

You can grant a user access to your ROSA cluster by adding them to the configured identity provider\.

The following procedure adds a user to a GitHub organization that’s configured for identity provisioning to the cluster\.

1. Navigate to [github\.com](https://github.com/) and log in to your GitHub account\.

1. Invite users that require ROSA cluster access to your GitHub organization\. For more information, see [Inviting users to join your organization](https://docs.github.com/en/organizations/managing-membership-in-your-organization/inviting-users-to-join-your-organization) in the GitHub documentation\.

## Step 9: Grant administrator permissions to a user<a name="getting-started-private-link-step-9"></a>

After you added a user to your configured identity provider, you can grant the user `cluster-admin` or `dedicated-admin` permissions for your ROSA cluster\.

### Configure `cluster-admin` permissions<a name="_configure_cluster_admin_permissions"></a>

1. Grant the `cluster-admin` permissions using the following command\. Replace `<idp_user_name>` and `<cluster_name>` with your user and cluster name\.

   ```
   rosa grant user cluster-admin --user=<idp_user_name> --cluster=<cluster_name>
   ```

1. Verify the user is listed as a member of the `cluster-admins` group\.

   ```
   rosa list users --cluster=<cluster_name>
   ```

### Configure `dedicated-admin` permissions<a name="_configure_dedicated_admin_permissions"></a>

1. Grant the `dedicated-admin` permissions with the following command\. Replace `<idp_user_name>` and `<cluster_name>` with your user and cluster name\.

   ```
   rosa grant user dedicated-admin --user=<idp_user_name> --cluster=<cluster_name>
   ```

1. Verify the user is listed as a member of the `cluster-admins` group\.

   ```
   rosa list users --cluster=<cluster_name>
   ```

## Step 10: Access a cluster through the web console<a name="getting-started-private-link-step-10"></a>

After you created a cluster administrator user or added a user to your configured identity provider, you can log in to your ROSA cluster through the web console\.

1. Obtain the console URL for your cluster using the following command\. Replace `<cluster_name>` with the name of your cluster\.

   ```
   rosa describe cluster -c <cluster_name> | grep Console
   ```

1. Navigate to the console URL in the output and log in\.
   + If you created a `cluster-admin` user, log in using the provided credentials\.
   + If you configured an identity provider for your cluster, choose the identity provider name in the **Log in with…​** dialog and complete any authorization requests presented by your provider\.

## Step 11: Deploy an application from the Developer Catalog<a name="getting-started-private-link-step-11"></a>

From the OpenShift Cluster Manager \(OCM\), you can deploy a Developer Catalog test application and expose it with a route\.

1. Navigate to [OpenShift Cluster Manager](https://console.redhat.com/openshift) and choose the cluster that you want to deploy the app into\.

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

   A new tab in the browser opens with a message that’s similar to the following\.

   ```
   Welcome to your Node.js application on OpenShift
   ```

1. \(Optional\) Delete the application and clean up resources\.

   1. In the **Administrator** perspective, choose **Home** > **Projects**\.

   1. Open the action menu for your project and choose **Delete Project**\.

## Step 12: Revoke administrator permissions and user access<a name="getting-started-private-link-step-12"></a>

You can revoke `cluster-admin` or `dedicated-admin` permissions from a user by using the ROSA CLI\.

To revoke access from a user, you must remove the user from your configured identity provider\.

### Revoke `cluster-admin` permissions from a user<a name="_revoke_cluster_admin_permissions_from_a_user"></a>

1. Revoke the `cluster-admin` permissions using the following command\. Replace `<idp_user_name>` and `<cluster_name>` with your user and cluster name\.

   ```
   rosa revoke user cluster-admin --user=<idp_user_name> --cluster=<cluster_name>
   ```

1. Verify that the user isn’t listed as a member of the `cluster-admins` group\.

   ```
   rosa list users --cluster=<cluster_name>
   ```

### Revoke `dedicated-admin` permissions from a user<a name="_revoke_dedicated_admin_permissions_from_a_user"></a>

1. Revoke the `dedicated-admin` permissions using the following command\. Replace `<idp_user_name>` and `<cluster_name>` with your user and cluster name\.

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

## Step 13: Delete a ROSA cluster and AWS STS resources<a name="getting-started-private-link-step-13"></a>

You can use the ROSA CLI to delete a ROSA cluster that uses the AWS Security Token Service \(AWS STS\)\. You can also use the ROSA CLI to delete the AWS Identity and Access Management \(IAM\) account\-wide roles, the cluster\-specific operator roles, and the OpenID Connect \(OIDC\) provider\. To delete the account\-wide inline and operator policies, you can use the IAM console\.

**Important**  
Account\-wide IAM roles and policies might be used by other ROSA clusters in the same account\. If they aren’t required by other clusters, you must only remove the resources\.

1. Delete the ROSA cluster and watch the logs using the following command\. Replace `<cluster_name>` with the name or ID of your cluster\.

   ```
   rosa delete cluster --cluster=<cluster_name> --watch
   ```
**Important**  
Before you remove the IAM roles, policies, and OIDC provider, you must wait for cluster to delete \. The account\-wide roles are required to delete the resources that are created by the installer\. The cluster\-specific operator roles are required to clean up the resources that are created by the OpenShift operators\. The operators use the OIDC provider to authenticate\.

1. Delete the OIDC provider that the cluster operators use to authenticate\.

   ```
   rosa delete oidc-provider -c <cluster_id> --mode auto
   ```

1. Delete the cluster\-specific operator IAM roles\.

   ```
   rosa delete operator-roles -c <cluster_id> --mode auto
   ```

1. Delete the account\-wide roles, replacing `<prefix>` with the prefix of the account\-wide roles to delete\. If you specified a custom prefix when creating the account\-wide roles, specify the default `ManagedOpenShift` prefix\.

   ```
   rosa delete account-roles --prefix <prefix> --mode auto
   ```

1. Delete the account\-wide inline and operator IAM policies\.

   1. Log in to the [IAM console](https://console.aws.amazon.com/iamv2/home#/home)\.

   1. On the left menu under **Access management**, choose **Policies**\.

   1. Select the account\-wide policy that you want to delete and choose **Actions** > **Delete**\.

   1. Enter the policy name and choose **Delete**\.

   1. Repeat this step to delete each of the account\-wide inline and operator policies for the cluster\.
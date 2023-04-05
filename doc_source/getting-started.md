# Getting started with ROSA<a name="getting-started"></a>

 ROSA is a turnkey application platform that provides a fully managed Red Hat OpenShift service deployed and operated on AWS\.

Many procedures of this user guide use the following command line tools:
+  **ROSA CLI** – A command line tool for working with ROSA clusters\. For more information, see the [Setting up the Rosa CLI](https://docs.openshift.com/rosa/rosa_cli/rosa-get-started-cli.html#rosa-setting-up-cli_rosa-getting-started-cli) topic in *the Red Hat documentation*\.
+  **OpenShift CLI** – A command line tool for creating applications and managing OpenShift Container Platform projects from a terminal\. For more information, see the [Getting started with the OpenShift CLI](https://docs.openshift.com/container-platform/4.10/cli_reference/openshift_cli/getting-started-cli.html) topic in *the Red Hat OpenShift CLI guide*\.
+  ** AWS CLI ** – A command line tool for working with AWS services, including ROSA\. For more information, see [Installing or updating the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) in *the AWS Command Line Interface User Guide*\. After installing the AWS CLI, we recommend that you also configure it\. For more information, see [Quick configuration with AWS configure](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config) in *the AWS CLI User Guide*\.

The following tutorials demonstrate the three different methods that you can use to configure and deploy your application to a ROSA cluster\. The recommended method for most customers to get started with ROSA is using AWS Security Token Service \(AWS STS\) short\-lived credentials for enhanced security protection\.

You can use AWS STS and the ROSA CLI to deploy public application workloads in a ROSA cluster\. You can also use AWS STS to deploy private application workloads in a private cluster using AWS PrivateLink\. For more information about AWS PrivateLink, see [What is AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html)\.

The ROSA CLI uses `auto` mode or `manual` mode to create the IAM resources required to provision a ROSA cluster\. `auto` mode immediately creates the required IAM roles and policies and an OpenID Connect \(OIDC\) provider\. `manual` mode outputs the AWS CLI commands that are needed to create the required IAM resources\. By using `manual` mode, you can review the generated AWS CLI commands before running them manually, or pass the commands to another administrator or group in your organization so they can create the resources\.

If you require the ROSA cluster and application workloads to be private, we recommend that you follow the getting started instructions for using ROSA with AWS PrivateLink\.

**Topics**
+ [Getting started with ROSA using AWS STS in auto mode](getting-started-sts-auto.md)
+ [Getting started with ROSA using AWS STS in manual mode](getting-started-sts-manual.md)
+ [Getting started with ROSA using AWS PrivateLink](getting-started-private-link.md)
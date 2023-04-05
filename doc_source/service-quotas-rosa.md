# ROSA service quotas<a name="service-quotas-rosa"></a>

 Red Hat OpenShift Service on AWS \(ROSA\) uses service quotas for Amazon EC2, Amazon Virtual Private Cloud \(Amazon VPC\), Amazon Elastic Block Store \(Amazon EBS\), and Elastic Load Balancing \(ELB\) to provision clusters\.

## Required minimum quotas for ROSA<a name="required-quotas"></a>

For the following Amazon EC2 and Amazon EBS quotas, ROSA requires a higher quota than the default service provides\. To use ROSA, you may need to request an increase for these quotas\. For more information, see [Requesting a quota increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-quota-increase.html) in the * Service Quotas User Guide*\.

**Important**  
For On\-Demand Standard \(A, C, D, H, I, M, R, T, Z\) Amazon EC2 instances, the default value of 5 vCPUs is not sufficient to create ROSA clusters\. ROSA requires 100 vCPUs or greater for cluster creation\. To increase this quota, open the [Service Quotas console](https://console.aws.amazon.com/servicequotas/home/services/ec2/quotas/L-1216C47A) and request a quota increase\.

**Note**  
You can check your quotas using the AWS SDKs, but the SDK calculation does not include existing ROSA resources\. The quota check in the SDK may pass, and ROSA cluster creation may fail\. To fix this issue, open the [Service Quotas console](https://console.aws.amazon.com/servicequotas) and request a quota increase\.


|  |  |  |  |  |  | 
| --- |--- |--- |--- |--- |--- |
|   **Name**   |   **Service code**   |   **Default**   |   **Minimum required**   |   **Adjustable**   |   **Description**   | 
|  Running On\-Demand Standard \(A, C, D, H, I, M, R, T, Z\) instances  |  ec2  |  5  |  100  |   [Yes](https://console.aws.amazon.com/servicequotas/home/services/ec2/quotas/L-1216C47A)   |  Maximum number of vCPUs assigned to the Running On\-Demand Standard \(A, C, D, H, I, M, R, T, Z\) instances\. The default value of 5 vCPUs is not sufficient to create ROSA clusters\. ROSA requires 100 vCPUs for cluster creation\.  | 
|  Storage for General Purpose SSD \(gp3\) volumes, in TiB  |  ebs  |  50  |  300  |   [Yes](https://console.aws.amazon.com/servicequotas/home/services/ebs/quotas/L-7A658B76)   |  The maximum aggregated amount of storage, in TiB, that can be provisioned across General Purpose SSD \(gp3\) volumes in this Region\. 300 TiB of storage is required for optimal performance\.  | 
|  Storage for General Purpose SSD \(gp2\) volumes, in TiB  |  ebs  |  50  |  300  |   [Yes](https://console.aws.amazon.com/servicequotas/home/services/ebs/quotas/L-D18FCD1D)   |  The maximum aggregated amount of storage, in TiB, that can be provisioned across General Purpose SSD \(gp2\) volumes in this Region\. 300 TiB of storage is required for optimal performance\.  | 
|  Storage for Provisioned IOPS SSD \(io1\) volumes, in TiB  |  ebs  |  50  |  300  |   [Yes](https://console.aws.amazon.com/servicequotas/home/services/ebs/quotas/L-FD252861)   |  The maximum aggregated amount of storage, in TiB, that can be provisioned across Provisioned IOPS SSD \(io1\) volumes in this Region\. 300 TiB of storage is required for optimal performance\.  | 

**Note**  
The default values are the initial quotas set by AWS, which are separate from the actual applied quota value and maximum possible service quota\. For more information, see [Terminology in Service Quotas](https://docs.aws.amazon.com/servicequotas/latest/userguide/intro.html#intro_getting-started) in the * Service Quotas User Guide*\.

## Default quotas for ROSA<a name="default-quotas"></a>

 ROSA uses the following default quotas for Amazon EC2, Amazon VPC, Amazon EBS, and Elastic Load Balancing\. For information on increasing quotas, see [Requesting a quota increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-quota-increase.html) in the * Service Quotas User Guide*\.

 Amazon EC2   
+  [EC2\-VPC Elastic IPs](https://docs.aws.amazon.com/general/latest/gr/ec2-service.html#limits_ec2) 

 Amazon VPC   
+  [VPCs per Region](https://docs.aws.amazon.com/general/latest/gr/vpc-service.html#vpc-quotas) 
+  [Network interfaces per Region](https://docs.aws.amazon.com/general/latest/gr/vpc-service.html#vpc-quotas) 
+  [Internet gateways per Region](https://docs.aws.amazon.com/general/latest/gr/vpc-service.html#vpc-quotas) 

 Amazon EBS   
+  [Snapshots per Region](https://docs.aws.amazon.com/general/latest/gr/ebs-service.html#limits_ebs) 
+  [IOPS for Provisioned IOPS SSD \(io1\) volumes](https://docs.aws.amazon.com/general/latest/gr/ebs-service.html#limits_ebs) 

 Elastic Load Balancing   
+  [Application Load Balancers per Region](https://docs.aws.amazon.com/general/latest/gr/elb.html#limits_elastic_load_balancer) 
+  [Classic Load Balancers per Region](https://docs.aws.amazon.com/general/latest/gr/elb.html#limits_elastic_load_balancer) 
# AWS Security Funtime

This is by no means enough to take the exam, but these were helpful points of reference that I used to review. 

The 3 most important tips, outside of reviewing the white papers and re:Invent videos:

1) try out every lab you can on ACloudGuru / LinuxAcademy, especially related to using CloudWatch and to using KMS
2) have a strong sense of how the following work together:
	- Lambda with everything
	- CloudFront, ELB (all 3 types), AWS WAF
	- CloudTrail, CloudWatch logs, S3, Athena
	- Macie with S3
3) have a good sense of the different use cases for the following:
- GuardDuty
- Inspector (especially what you can and can’t choose, and how to troubleshoot)
- Config (especially types of config rules)
- Trusted Advisor (know its limitations)

I cannot stress the importance of trying these tools out, going through labs, and stubbing your toe. Even if you have experience with some of them, you need exposure to all in order to pass. People who just try to cram for this routinely fail, it was designed to beat them. Besides, learning from labs is much more efficient than cramming, and you get immediate feedback if you make a mistake.

## AWS CloudTrail
- [AWS CloudTrail FAQ](https://aws.amazon.com/cloudtrail/faqs/) x 3
- CloudTrail provides event history of your AWS account activity, including actions taken through the AWS Management Console, AWS SDKs, command line tools, and other AWS services.
- It is recommended to use a dedicated S3 bucket for CloudTrail logs. 
- CloudTrail can also send logs to CloudWatch Logs, which can then trigger CloudWatch Events
![CloudTrail flow](Product-Page-Diagram-AWSX-CloudTrail_How-it-Works.d2f51f6e3ec3ea3b33d0c48d472f0e0b59b46e59.png)
### Multiple accounts
- Within an AWS Organization, you can create one CloudTrail to cover all accounts.
![CloudTrail for Orgs](organization-trail.png)
### Management and data events
- Management and Data events are handled by separate CloudTrails. 
  - **Data Events**: operations on or within a resource
		- Often high-volume
		- Disabled by default
		- You must explicitly add the supported resources or resource types for which you want to collect activity to a trail.
  - **Management Events**: Configuration or security changes
  - You should log the events to separate buckets, then configure access to the CloudTrail and read only access to the S3 bucket using an IAM policy attached to the user or group. 
### Regions
- When you apply a trail to all regions, CloudTrail uses the trail that you create in a particular region to create trails with identical configurations in all other regions in your account. 
### Integrity
- To determine whether a log file was modified, deleted, or unchanged after CloudTrail delivered it, you can use CloudTrail log file integrity validation.


## AWS KMS
- [KMS FAQ](https://aws.amazon.com/kms/faqs/) x 2
### CMK
- Imported key material
  - Automatic key rotation is not available for CMKs that have imported key material, you will need to do this manually.
- Customer managed keys
  - A customer managed CMK supports automatic key rotation once per year. 
  - Creating and managing your own CMK gives you more flexibility, including the ability to create, rotate, disable, and define access controls, and to audit the encryption keys used to protect your data. 
- AWS managed keys 
  - AWS managed keys automatically rotate once every three years.

## AWS Config
- [AWS Config FAQ](https://aws.amazon.com/config/faq/) x 2
- You can send notifications or take automated action with Lambda when a resource violates a rule.
![How Config Works](how-AWSconfig-works.png)

## Amazon Inspector
- [Inspector FAQ](https://aws.amazon.com/inspector/faqs/) x 2
- The runtime behavior package checks for insecure protocols like Telnet, FTP, HTTP, IMAP, rlogin etc. 
- Neither the AWS Config restricted-common-ports check or Trusted Advisor will give you this information.

## AWS CloudWatch
- [CloudWatch FAQ](https://aws.amazon.com/cloudwatch/faqs/) x 2
- You can use Amazon CloudWatch Logs to monitor, store, and access your log files from EC2 instances, AWS CloudTrail, Route 53, and other sources. 
- CloudWatch alone lacks the business rules that are provided with GuardDuty to create an event whenever malicious or unauthorized behavior is observed.
	- GuardDuty can trigger CloudWatch Events which can then be used for a variety of activities like notifications or automatically responding to a threat.
- If an anomaly is detect, CloudWatch Event can trigger a Lambda.
### CloudWatch Logs
- You can use Amazon CloudWatch Logs to monitor, store, and access your log files from EC2 instances, AWS CloudTrail, Route 53, and other sources. 
- You can then retrieve the associated log data from CloudWatch Logs.
### CloudWatch Events
- You can use CloudWatch Events to schedule automated actions that self-trigger at certain times using cron or rate expressions.
- You can configure Amazon Inspector as a target for CloudWatch Events. 

## AWS Lambda
- [Lambda FAQ](https://aws.amazon.com/lambda/faqs/) x 2
- For Lambda to send logs to CloudWatch, the function execution role needs to permission to write to CloudWatch.
- For Lambda to make API calls to DynamoDB, the function execution role needs many permissions to interact with DynamoDB.
```json
{
	"Version": "2012-10-17",
	"Statement": [{
			"Effect": "Allow",
			"Action": [
				"dynamodb:BatchGetItem",
				"dynamodb:GetItem",
				"dynamodb:Query",
				"dynamodb:Scan",
				"dynamodb:BatchWriteItem",
				"dynamodb:PutItem",
				"dynamodb:UpdateItem"
			],
			"Resource": "arn:aws:dynamodb:eu-west-1:123456789012:table/SampleTable"
		},
		{
			"Effect": "Allow",
			"Action": [
				"logs:CreateLogStream",
				"logs:PutLogEvents"
			],
			"Resource": "arn:aws:logs:eu-west-1:123456789012:*"
		},
		{
			"Effect": "Allow",
			"Action": "logs:CreateLogGroup",
			"Resource": "*"
		}
	]
}
```
![Lambda architecture](lambda-role.png)

## AWS Trusted Advisor
- Checks Security Groups for rules that allow unrestricted access (0.0.0.0/0) to specific ports such as SSH. 
  - [AWS Config](#aws-config) can alert you to any _modifications_ to a Security Group but out of the box, it will not perform a check for _unrestricted access_.
  - Ports flagged green are typically used by applications that require unrestricted access, such as HTTP and SMTP.
- Other core best practice checks:
  - S3 Bucket Permissions
  - IAM Use
  - MFA on Root Account
  - EBS Public Snapshots
  - RDS Public Snapshots

## AWS VPC
- [VPC FAQ](https://aws.amazon.com/vpc/faqs/)
### Flow Logs
- VPC Flow Logs enables you to capture information about the IP traffic going to and from network interfaces in your VPC. 
- Flow log data can be published to Amazon CloudWatch Logs and Amazon S3.
- Flow logs can help you with a number of tasks:
  - Diagnosing overly restrictive security group rules
  - Monitoring the traffic that is reaching your instance
  - Determining the direction of the traffic to and from the network interfaces
  - Aiding in investigating suspicius traffic
### DNS servers
- The default DHCP option set specifies AmazonProvidedDNS but you can provide the IP address of up to 4 of your own DNS servers. 
- You cannot update the existing option set, you must delete it and create a new one.
### Security groups
- Security groups are stateful, if you have allowed the inbound traffic you do not need to create a rule to allow the outbound reply. 
- By default an SG allows any outbound traffic so you don't need to add an outbound rule to a server in a public subnet.
### AWS Direct Connect
- With AWS Direct Connect plus VPN, you can combine one or more AWS Direct Connect dedicated network connections with the Amazon VPC VPN. 
- This combination provides an IPsec-encrypted private connection that also reduces network costs, increases bandwidth throughput, and provides a more consistent network experience than internet-based VPN connections.
![AWS Direct Connect and VPN](aws-direct-connect-vpn.png)

## AWS GuardDuty
- [GuardDuty FAQ](https://aws.amazon.com/guardduty/faqs/)
- It is a managed service that can watch CloudTrail, VPC Flow Logs and DNS Logs, watching for malicious activity. 
- It can detect instances exhibiting signs of compromise, such as: 
  - Attempting to communicate with a command and control server.
  - Behaving as a spam bot with email traffic over port 25.
  - Sending requests that look like it is part of a DoS attack
  - [GuardDuty Backdoor](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_backdoor.html)
- It has a build-in list of suspect IP addresses and you can also upload your own lists of IPs.
- GuardDuty can trigger CloudWatch Events which can then be used for a variety of activities like notifications or automatically responding to a threat

## AWS Systems Manager 
### Parameter Store
- Services that support parameter store:
  - Amazon EC2
  - Amazon ECS
  - AWS Lambda
    - If a service does not directly support it (e.g., RDS), just **use Lambda in association with the service**
  - AWS CloudFormation
  - AWS CodeBuild
  - AWS CodeDeploy
- Configure integration with the following AWS services for encryption, notification, monitoring, and auditing:
  - Amazon SNS
  - Amazon CloudWatch
  - AWS CloudTrail
  - AWS KMS
    - An instance role needs permission both to read an SSM parameter and to use KMS to decrypt it.
    - Parameter Store uses KMS customer master keys to encrypt the parameter values when you create or change them.
    - Parameter Store supports only symmetric CMKs. You cannot use asymmetric CMKs to encrypt your parameters.
![Standard Parameter Encryption](service-pstore-standard.png)

### Patch Manager
- The default predefined patch baseline for Windows servers in Patch Manager is `AWS-DefaultPatchBaseline`.

## AWS ElastiCache
- Supports encryption only for Redis 3.2.6, 4.0.10 and later, not Memcached.

## AWS IAM
### Account Compromise
- If you suspect that your account is compromised, do the following:
  - Change your AWS account root user password.
  - Rotate and delete all root and AWS Identity and Access Management (IAM) access keys.
  - Delete any potentially compromised IAM users, and change the password for all other IAM users.
  - Delete any resources on your account you didn't create, such as EC2 instances and AMIs, EBS volumes and snapshots, and IAM users.
  - Respond to any notifications you received from AWS Support through the AWS Support Center.
### Policy evaluation logic
1. The AWS service receives the request
2. AWS first authenticates the principal.
    - Except for services such as S3 that allow anonymous access). 
3. Next, AWS determines which policy to apply to the request. 
    - Actions (or operations) – The actions or operations that the principal wants to perform.
    - Resources – The AWS resource object upon which the actions or operations are performed.
    - Principal – The user, role, federated user, or application that sent the request. Information about the principal includes the policies that are associated with that principal.
    - Environment data – Information about the IP address, user agent, SSL enabled status, or the time of day.
    - Resource data – Data related to the resource that is being requested. This can include information such as a DynamoDB table name or a tag on an Amazon EC2 instance.
4. Then, AWS evaluates the policy types and arranges an order of evaluation. 
    - Identity-based policies
    - Resource-based policies
    - IAM permissions boundaries
    - AWS Organizations service control policies (SCPs)
    - Session policies (e.g. for federated user sessions)
5. Finally, AWS then processes the policies against the request context to determine if it is allowed.
![Policy Evaluation Diagram](PolicyEvaluationHorizontal.png)
### ADFS
![ADFS flow](adfs.png)
1. Corporate user accesses the corporate Active Directory Federation Services portal sign-in page and provides Active Directory authentication credentials.
2. AD FS authenticates the user against Active Directory.
3. Active Directory returns the user’s information, including AD group membership information.
4. AD FS dynamically builds ARNs by using Active Directory group memberships for the IAM roles and user attributes for the AWS account IDs, and sends a signed assertion to the users browser with a redirect to post the assertion to AWS STS.
5. Temporary credentials are returned using STS AssumeRoleWithSAML.
6. The user is authenticated and provided access to the AWS management console.

## AWS EC2
- If you connect to your instance using SSH and get any of the following errors, "Host key not found in `[directory]`", "Permission denied (publickey)", or "Authentication failed, permission denied", verify that you are connecting with the appropriate user name for your AMI *and* that you have specified the proper private key (.pem) file for your instance.
- If you lose the private key for an EBS-backed instance, you can regain access to your instance. You must: 
1. stop the instance, 
2. detach its root volume and attach it to another instance as a data volume, 
3. modify the `authorized_keys` file, 
4. move the volume back to the original instance, and 
5. restart the instance.

## AWS Marketplace
### IDS/IPS
- AWS GuardDuty is not an IDS. While it does perform _threat_ detection based on logs, it does not detect _intrusion_. 
- AWS Shield is not an IPS. It mitigates DDoS attacks, but it does not prevent intrusion.
- AWS acknowledge that they do not provide IPS/IDS. 
  - Instead they suggest that third-party software can be used to provide additional functionality such as deep packet inspection, IPS/IDS, or network threat protection. 
  - Search for IPS on AWS Marketplace and you will find a range of suitable products!

## Athena
- [Athena FAQ](https://aws.amazon.com/athena/faqs/)

## AWS Shield
- [AWS Shield FAQ](https://aws.amazon.com/shield/faqs/)

## AWS CloudFront
- [CloudFront FAQ](https://aws.amazon.com/cloudfront/faqs/)

## Route 53
- [R53](https://aws.amazon.com/route53/faqs/)

## Artifact
- [Artifact FAQ](https://aws.amazon.com/artifact/faq/)

## Cognito
- [https://aws.amazon.com/cognito/faqs/](https://aws.amazon.com/cognito/faqs/)

## CloudHSM
- [https://aws.amazon.com/cloudhsm/faqs/](https://aws.amazon.com/cloudhsm/faqs/)

## Organizations
- [https://aws.amazon.com/organizations/faqs/](https://aws.amazon.com/organizations/faqs/)

## ACM
- [https://aws.amazon.com/certificate-manager/faqs/](https://aws.amazon.com/certificate-manager/faqs/)

## EC2 Autoscaling
- [https://aws.amazon.com/ec2/autoscaling/faqs/](https://aws.amazon.com/ec2/autoscaling/faqs/)

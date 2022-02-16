## aws-blog-integrate-administrator-approval-ec2imagebuilder-awssystemmanager

This repository contains two CloudFormation templates that support an AWS Blog Post:
- simple_web_server_pipeline.yml: a non production example of EC2 Image builder pipeline that could create a Tomcat Server image.
- system_manager_automation_document.yml: a non production automation document that orchestrates creation of the Tomcat Server image, integrating administrator approval and resources sharing.
Since there is resource sharing involved, you must be careful on how you define your resources, and check inputs in order to avoid sharing a resources with an unauthorized principal.

# simple_web_server_pipeline.yml
# Description
This CloudFormation template creates an EC2 Image Builder pipeline. When launched, the pipeline will build an Amazon Machine Image (AMI) that will be tested for vulnerabilities with Amazon Inspector. Be aware that the AMI is not production ready, and is provided as an example  of how you could create a Tomcat Server within a Golden AMI process. Note that all created resources that support tags will have the tags owner/golden-ami-automation. This tag is also used in Roles, when defining Conditions. If you wish to modify this tag, make sure to change all references to it.


## Parameters
The list of parameters for this template:

### CustomSubnetId 
Type: String  
Description: If you do not have a default VPC, or want to use a different VPC, specify the ID of a subnet in which to place the instance used to customize your EC2 AMI. If not specified, a subnet from your default VPC will be used. 
### CustomSecurityGroupId 
Type: CommaDelimitedList  
Description: Required if you specified a custom subnet ID. Comma-delimted list of one or more IDs of security groups belonging to the VPC to associate with the instance used to customize your EC2 AMI. 
### BuildInstanceType 
Type: CommaDelimitedList 
Default: t2.micro 
Description: Comma-delimited list of one or more instance types to select from when building the image. Image Builder will select a type based on availability. The supplied default is compatible with the AWS Free Tier. 

## Resources
The list of resources this template creates:

### InstanceRole 
Type: AWS::IAM::Role  
### InstanceProfile 
Type: AWS::IAM::InstanceProfile  
### TomcatImageInfrastructureConfiguration 
Type: AWS::ImageBuilder::InfrastructureConfiguration  
### InstallTomcatComponent 
Type: AWS::ImageBuilder::Component  
### TomcatServerImageRecipe 
Type: AWS::ImageBuilder::ImageRecipe  
### AmazonLinux2WithTomcat 
Type: AWS::ImageBuilder::ImagePipeline  

## Outputs
The list of outputs this template exposes:



# system_manager_automation_document.yml
# Description
This CloudFormation template creates a System Manager Automation runbook that will orchestrate a process to integrate administrator approval in an EC2 Image builder pipeline, before sharing the outputs with selected principals. Be aware that the Template is not production ready. You need to remove the (optional) steps that you don't need.  Also, notice that there is no mechanims in this automation document to prevent sharing with unauthorized principals.  You must handle this before using that template in production. Ideas are discussed in the associated blog post. Note that some IAM actions are defined with the condition "StringEquals:'aws:ResourceTag/owner': golden-ami-automation" in order to further control access.  Make sure to add this tag to the SNS Topic and the RAM resources share that you provide. If you wish to change the take, ensure you replace all mention of this tag in this template.


## Parameters
There is not parameters for this template. However, the created automation document exposes parameters:

#### ImageBuilderPipelineArn:
Description: (Required) Corresponding EC2 Image Builder Pipeline to execute.
Type: String
#### PipelineApproverArn:
Description: (Required) Arn of the Role that must approve the built image.
Type: StringList
#### SnsNotificationArn:
Description: (Required) Arn of the SNS topics for notifications.
Type: String
#### ResourceShareArn:
Description: (Optional) Arn of the resource share to use to share EC2 Builder images. Used in Step 7. If not necessary, this parameter must be removed along with the corresponding step.
Type: String
#### AMIPrincipals:
Description: (Optional) The EC2 AMI will be shared with this list of principals. Enter Account IDs. Used in Step 8. If not necessary, this parameter must be removed along with the corresponding step.
Type: StringList

## Resources
The list of resources this template creates:

### AutomationRole 
Type: AWS::IAM::Role  
### EC2BuilderAutomation 
Type: AWS::SSM::Document  

## Outputs
There is not output


## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.


# Stepfunctions SSM Windows Patching
AWS step function and associated Lambda functions to patch Windows servers by environment and solution in different availability zones using SSM automation.

## Written to meet the following requirements:
- Patching Instances per availability zone. AZ A then AZ B and C. 
- Patching Data tagged instances before Application instances to ensure Datasources are available on Application launch.
- Utilisation of SSM document which allows for custom pre or post patching scripts.
- AMI of Instances pre-patch.
- Retrying of any automation failures, email notification on retry.
- Email on completion or failure.

## Installation Notes
- Download files 
- Upload zips to S3 bucket, please place zips in root of bucket.
- Run deployment.yml and specify bucket name and contactTo and contactFrom email identities. Please note, the bucket must be in same region as Lambda functions.
- If using new contactTo and contactFrom email addresses, Please ensure you verify the SES US East 1 contactTo and contactFrom email addresses! The ssm_patchingnotfications will create the SES email identity but cannot verify.

## General information
Includes the following artifacts
- **deployment.yml** A Cloudformation YAML template to deploy the Stepfunction, Lambda functions, SSM document and IAM roles.
- **stepExecutionExample.json** A example JSON statement to be passed into the Step Function to initiate the function.
- **ssm_patchautomation.zip** A Python 3.6 Lambda function to filter instances based on tag and create SSM automation jobs for those instances.
- **ssm_patchingnotifications.zip** Function to send success or failure emails.
- **ssm_retry.zip** Function to retry failed automations. The Step function will retry a failed function up to 3 times.
- **ssm_patchpolling.zip** Function to poll the status of automations, runs every 3 minutes while Step Function is active.

## How it works
The AWS Tag with a Key of **instancedata** with the below value is added to all instances and modified based on Solution, Environment and Instance Role.

{"solution":"ClientA","environment":"production","instance":"Web Server"}

This allows for filtering of different environment and solutions within the same AWS account. Then customise your EC2 instances with the correct corresponding tag. The following values for *instance* in the above JSON are matched against the below strings in the ssm_patchautomation function:
- SQL Server
- Postgresql Server
- File Server
- Application Server
- Windows Server
- Web Server

So for example, a development SQL Server would be tagged Key: instancedata Value: {"solution":"ClientA","environment":"production","instance":"SQL Server"}

More strings can be added as per requirements. **The strings are case sensitive**

Once all instances are tagged you can then commence patching by executing the step function.

## Executing the Step Function
The Step Function will take the following JSON as input. Please customise this with relevant solution, environment and region.

{
  "patchinginfo": {
    "solution": "ClientA",
    "environment": "production",
    "region": "us-east-1"
  }
}

Pass in the relevant values that match the instancedata tag.




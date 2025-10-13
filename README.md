# Automated S3 Remediation to Enforce Block Public Access
This project demonstrates how to detect and automatically remediate non-compliant Amazon S3 buckets that have Block Public Access disabled.
You’ll use AWS Config to create a rule and configure automated remediation that reverts any unauthorized or accidental changes, ensuring your buckets remain private and compliant by default..

# Scenario
In this walkthrough, we’ll implement automated enforcement of S3 Block Public Access settings across all buckets in an AWS account.
If Block Public Access is disabled on any bucket — whether by accident or intentionally — AWS Config will automatically detect the non-compliance and restore secure settings.

As a security best practice, S3 buckets should remain private unless there is a clear use case for public access (e.g., hosting a static website).
For production environments, consider maintaining public and private buckets in separate accounts and enforcing policies at the account level.
In this lab, we’ll focus on bucket-level enforcement for demonstration purposes.

<img width="739" height="356" alt="image" src="https://github.com/user-attachments/assets/bac447b1-6bcd-401b-80b7-acbe062ed2de" />

# Step 1: Create a Non-Compliant S3 Bucket
1. Navigate to Amazon S3 → Create bucket.

2. Give your bucket a unique name, for example:

       demo-noncompliant-bucket-222

<img width="542" height="214" alt="image" src="https://github.com/user-attachments/assets/c070509b-57a9-43cc-ac70-f328f352e1e3" />

<img width="959" height="349" alt="image" src="https://github.com/user-attachments/assets/88403378-64cb-4da5-a7b1-20cbbab12b21" />

3. Under Block Public Access settings, uncheck all options to disable Block Public Access.

<img width="916" height="266" alt="image" src="https://github.com/user-attachments/assets/fba1c8ab-e510-4a07-9c69-3a394aa44163" />

4. Confirm the warning and create the bucket.

You now have a non-compliant bucket that will trigger our AWS Config rule later.

<img width="935" height="239" alt="image" src="https://github.com/user-attachments/assets/c6cd3bfb-dde9-4dcc-a6dc-b7db38678fe5" />

# Step 2: Create an AWS Config Rule
1. Open AWS Config and go to Rules → Add rule.

2. Choose AWS Managed Rule and search for:

          s3-bucket-level-public-access-prohibited

<img width="722" height="367" alt="image" src="https://github.com/user-attachments/assets/3c7d431a-4369-44c3-83cf-91bead641e22" />

Under Evaluation mode, select:

• Scope of changes: Resources

• Resource type: AWS::S3::Bucket

You may leave the excludedPublicBuckets parameter empty (best practice: all buckets should be private).

<img width="702" height="164" alt="image" src="https://github.com/user-attachments/assets/98f037f5-55e4-4e01-803e-494dd37bd3de" />

Review and create the rule.

<img width="959" height="278" alt="image" src="https://github.com/user-attachments/assets/9d348c91-a1a2-4269-b8c4-8ca41a934b54" />

# Step 3: Enable Automated Remediation

1. After the rule is created, select it and choose:
   
2. Actions → Manage remediation → Automatic remediation.

For Remediation action, search and select:

       AWSConfigRemediation-ConfigureS3BucketPublicAccessBlock

<img width="770" height="254" alt="image" src="https://github.com/user-attachments/assets/661b120a-03b7-4995-b78f-788c62d01f16" />

Under Resource ID parameter, select BucketName.

3. In the Parameters section:

• Set all options (BlockPublicAcls, IgnorePublicAcls, BlockPublicPolicy, RestrictPublicBuckets) to true.

• Specify the AutomationAssumeRole ARN, e.g.:

       arn:aws:iam::<account-id>:role/AutomatedS3Remediation

<img width="698" height="283" alt="image" src="https://github.com/user-attachments/assets/ced3ec69-a9cb-4f86-ab55-ed1f11301325" />

About the role we need
I would normally need to create this role in this account if I was using my AWS account, but this role was pre created in this lab environment. 
You might be wondering though: how do I know what permissions to grant this role when creating it?

Luckily, if I wanted to create this role, AWS spells it out for us. Search for “AWSConfigRemediation-ConfigureS3BucketPublicAccessBlock” with your search engine and you’ll find this link (https://docs.aws.amazon.com/systems-manager-automation-runbooks/latest/userguide/automation-aws-block-public-s3.html).

Towards the bottom, you’ll see “Required IAM permissions” which tells us we need:

ssm:StartAutomationExecution

ssm:GetAutomationExecution

s3:GetAccountPublicAccessBlock

s3:PutAccountPublicAccessBlock

s3:GetBucketPublicAccessBlock

s3:PutBucketPublicAccessBlock

In terms of the JSON IAM policy, this is what would translate to:

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "ssm:StartAutomationExecution",
                    "ssm:GetAutomationExecution"
                ],
                "Resource": "*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:GetAccountPublicAccessBlock",
                    "s3:PutAccountPublicAccessBlock"
                ],
                "Resource": "*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:GetBucketPublicAccessBlock",
                    "s3:PutBucketPublicAccessBlock"
                ],
                "Resource": "arn:aws:s3:::*"
            }
        ]
    }

# Step 4: Create or Verify the IAM Role
But anyway, this role was created for me, and this will be my ARN:

    arn:aws:iam::<account-id>:role/AutomatedS3Remediation

Look in the top right corner for your AWS Account ID. You can click on the username and then copy the ID value. Put that in to the ARN and you’re good to go. So for example, this is mine:

<img width="687" height="270" alt="image" src="https://github.com/user-attachments/assets/926ddd07-3665-41ae-997b-875002db298d" />

Then save and create your automated remediation action.

<img width="757" height="191" alt="image" src="https://github.com/user-attachments/assets/aed384e9-4566-4603-84af-e24c844961c3" />

# Step 5: Test the Automated Remediation
1. Return to the AWS Config rule page.

2. Scroll to Resources in scope — your S3 bucket should appear as Non-compliant.

3. Wait a few minutes for AWS Config to trigger remediation automatically (10–30 minutes).

4. To test instantly, you can manually select the resource → Remediate.

5. Once complete, the resource status will show Action executed successfully and Compliant.

6. Verify in S3 that Block Public Access is now re-enabled for your bucket.
<img width="707" height="170" alt="image" src="https://github.com/user-attachments/assets/e72fdb17-3f34-4946-84eb-6dc2b82c1b26" />

<img width="689" height="165" alt="image" src="https://github.com/user-attachments/assets/de8ee191-9428-48a6-8b46-f4d92e750b2f" />

Eventually, you will see:
The action status to change until it eventually says “Action executed successfully”

<img width="695" height="186" alt="image" src="https://github.com/user-attachments/assets/a31e5c40-39f9-49f5-9c06-c7786e67120b" />

Finally, let’s head back over to S3 and check the Block public access setting – we should see it set to On.

<img width="932" height="314" alt="image" src="https://github.com/user-attachments/assets/5a49470b-3e4f-4953-adc6-56abce8f5f8b" />

Conclusion
This setup now ensures that if anyone ever tries (accidentally or maliciously) to disable “Block Public Access” for one of your S3 buckets in this account, it will automatically revert back in only a few minutes maintaining data’s confidentiality and compliance posture.





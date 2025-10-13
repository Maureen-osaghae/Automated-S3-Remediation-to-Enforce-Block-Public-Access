# Automated S3 Remediation to Enforce Block Public Access
This project demonstrates how to detect and automatically remediate non-compliant Amazon S3 buckets that have Block Public Access disabled.
You’ll use AWS Config to create a rule and configure automated remediation that reverts any unauthorized or accidental changes, ensuring your buckets remain private and compliant by default..

# Scenario
In this scenario, I’ll implement automated enforcement of S3 Block Public Access settings across S3 buckets within an AWS account. This automation ensures that if Block Public Access is disabled on any bucket—whether through accidental changes or unauthorized actions—AWS Config will automatically detect the non-compliant configuration and restore the secure settings.

<img width="739" height="356" alt="image" src="https://github.com/user-attachments/assets/bac447b1-6bcd-401b-80b7-acbe062ed2de" />
As a security best practice, buckets should always start off completely private. Only in specific use cases should buckets be made public, such as if you are deploying a public static website or sharing public objects.

Even then, you should always keep private buckets and public buckets in entirely different AWS accounts. This helps prevent misconfiguration, and it also enables you to enforce settings like Block Public Access at the account level instead of the bucket level.

However, in this lab, we’ll explore Block Public Access at the bucket level for demonstration purposes. The concept generally translates to account level as well.

# Create a noncompliant S3 bucket
Let’s start the lab by creating a non-compliant S3 bucket. Since we’re going to be checking for Block Public Access non-compliance, we’ll want to create a bucket that disables this feature.

<img width="542" height="214" alt="image" src="https://github.com/user-attachments/assets/c070509b-57a9-43cc-ac70-f328f352e1e3" />

I’ll name my bucket demo-noncompliant-bucket-222 but you should use something unique.

<img width="959" height="349" alt="image" src="https://github.com/user-attachments/assets/88403378-64cb-4da5-a7b1-20cbbab12b21" />


Disable “Block Public Access” by unchecking the checkbox and confirming.

<img width="916" height="266" alt="image" src="https://github.com/user-attachments/assets/fba1c8ab-e510-4a07-9c69-3a394aa44163" />

Create your bucket.

<img width="935" height="239" alt="image" src="https://github.com/user-attachments/assets/c6cd3bfb-dde9-4dcc-a6dc-b7db38678fe5" />

# Creating our Config rule
Now, let’s navigate over to AWS Config and let’s create a new rule.

<img width="652" height="200" alt="image" src="https://github.com/user-attachments/assets/d6eb4a14-b7c5-41f9-9bba-63be73411024" />

Config should already be enabled for your account, so you can head straight to “Rules.” Then Add rule.

Select Add AWS managed rule so that we don’t have to create it from scratch, and then search for s3-bucket-level-public-access-prohibited.

<img width="722" height="367" alt="image" src="https://github.com/user-attachments/assets/3c7d431a-4369-44c3-83cf-91bead641e22" />

Under “Evaluation mode” we’ll want to select Resources for the scope of changes, since we’re looking for changes to AWS resources.

For the “Resource category” we could leave it at All resource categories or we can narrow it down to AWS resources, it won’t make a difference here.

Then, for the “Resource type” we can search for and select AWS S3 Bucket but it will probably already be selected for you based on the rule you selected.

<img width="901" height="293" alt="image" src="https://github.com/user-attachments/assets/964b00ef-9d2e-4a59-91b1-51b74dfbec78" />

<img width="702" height="164" alt="image" src="https://github.com/user-attachments/assets/98f037f5-55e4-4e01-803e-494dd37bd3de" />

Below that, the excludedPublicBuckets parameter would be useful if you wanted to exclude S3 buckets from this rule’s scope. So if you wanted to have a public bucket, you could use this. But again, best practice dictates you should have public buckets in entirely different accounts, so you should never really be using this for this type of rule.

<img width="959" height="278" alt="image" src="https://github.com/user-attachments/assets/9d348c91-a1a2-4269-b8c4-8ca41a934b54" />

# Enable automated remediation
After creating your rule, select it and click on Actions -> Manage remediation. Select Automatic remediation.

<img width="770" height="254" alt="image" src="https://github.com/user-attachments/assets/661b120a-03b7-4995-b78f-788c62d01f16" />

Leave the defaults unless otherwise instructed.

For the “Choose remediation action” option, search for and select AWSConfigRemediation-ConfigureS3BucketPublicAccessBlock

(Pro tip, you can just start searching for ConfigureS3BucketP and it will filter down)

<img width="635" height="110" alt="image" src="https://github.com/user-attachments/assets/684c5bd8-a5f8-4772-b8a5-3dd4435501c9" />

Familiarize yourself with what this action does and the available input parameters. In our case, we’ll want to use all of the defaults, but we need to pass in the non-compliant bucket name ad we’ll also need to provide an AutomationAssumeRole ARN.

This instructs Config to automatically pass in the resource ID of the noncompliant resource as a parameter of the remediation action, that way the automation knows which bucket to modify.

Scroll down until you see “Resource ID parameter.” Click on the dropdown and select BucketName.

<img width="808" height="174" alt="image" src="https://github.com/user-attachments/assets/0a125c68-1a9a-4396-b85c-2fa6e2ecc953" />

Then, below that, we have a “Parameters” section. You’ll notice the BucketName portion is already filled in for you and you can’t modify it.

For the other options, we’ll want to put true for everything except the final one, which is the AutomationAssumeRole input. This needs to be the role ARN that enables the automation to perform actions on your behalf. Let’s talk about that.

<img width="698" height="283" alt="image" src="https://github.com/user-attachments/assets/ced3ec69-a9cb-4f86-ab55-ed1f11301325" />

About the role we need
You would normally need to create this role in your account, but we’ve already created this for you to shorten the lab a little bit.

You might be wondering though: how do I know what permissions to grant this role when creating it?

Luckily, AWS spells it out for us. Search for “AWSConfigRemediation-ConfigureS3BucketPublicAccessBlock” with your search engine and you’ll find this link(https://docs.aws.amazon.com/systems-manager-automation-runbooks/latest/userguide/automation-aws-block-public-s3.html). (It’s easy to misclick as there’s another one similarly named but it’s for block public access at the account level, which is a valid one but not what we’re doing specifically in this lab).

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

# Finishing setup
But anyway, this role was created for me, and this will be my ARN:

    arn:aws:iam::<account-id>:role/AutomatedS3Remediation

Look in the top right corner for your AWS Account ID. You can click on the username and then copy the ID value. Put that in to the ARN and you’re good to go. So for example, this is mine:

<img width="687" height="270" alt="image" src="https://github.com/user-attachments/assets/926ddd07-3665-41ae-997b-875002db298d" />

Then save and create your automated remediation action.

<img width="757" height="191" alt="image" src="https://github.com/user-attachments/assets/aed384e9-4566-4603-84af-e24c844961c3" />

# Testing the automated remediation
You should be on your rule page (the rule we just created). If not, navigate there.

Then, scroll down all the way at the bottom until you see “Resources in scope.” You should already see your S3 bucket, but if not, give it about a minute for Config to discover it.

<img width="707" height="170" alt="image" src="https://github.com/user-attachments/assets/e72fdb17-3f34-4946-84eb-6dc2b82c1b26" />

<img width="689" height="165" alt="image" src="https://github.com/user-attachments/assets/de8ee191-9428-48a6-8b46-f4d92e750b2f" />

We can also go up and click on Actions -> Re-evaluate to speed it along, as this can take a few minutes, but you don’t need to do that if you already see the bucket as “noncompliant.”

The next step is to wait and see Config automatically remediate the issue! This can take 10-30 minute. I just did it myself and it took 23 minutes. You can speed it along if you’d like, especially since you have limited time in the lab environment, by selecting the resource and clicking on Remediate. Normally when enabling automated remediation you don’t need to take this action manually and you may not even have permissions to directly take that action (as it instead executes through a role), but we granted you permissions in the lab so you could speed it along if you wanted.

<img width="695" height="186" alt="image" src="https://github.com/user-attachments/assets/a31e5c40-39f9-49f5-9c06-c7786e67120b" />

Eventually, you will see:
The action status to change until it eventually says “Action executed successfully”

Let’s change the filter to “All” or “Compliant” and we should see our bucket. Let’s click on it. Then click on “Resource timeline.”

<img width="678" height="212" alt="image" src="https://github.com/user-attachments/assets/e82b8ae2-f5e4-4f28-b585-2907eac68ff8" />

Finally, let’s head back over to S3 and check the Block public access setting – we should see it set to On.

<img width="932" height="314" alt="image" src="https://github.com/user-attachments/assets/5a49470b-3e4f-4953-adc6-56abce8f5f8b" />

Conclusion
This setup now ensures that if anyone ever tries (accidentally or maliciously) to disable “Block Public Access” for one of your S3 buckets in this account, it will automatically revert back in only a few minutes. This is a great use case of when automated remediation is both practical and helpful.










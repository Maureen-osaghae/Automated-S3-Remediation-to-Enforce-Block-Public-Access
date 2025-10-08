# Automated-S3-Remediation-to-Enforce-Block-Public-Access
Learn how to detect and automatically remediate non-compliant S3 buckets that have Block Public Access disabled. This lab will teach you how to create an AWS Config rule and configure automated remediation to automatically revert any unauthorized or accidental changes that disable Block Public Access on S3 buckets. This ensures that buckets remain private by default, improving security posture and preventing unintended public exposure of data.

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






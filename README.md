# Automated-S3-Remediation-to-Enforce-Block-Public-Access
Learn how to detect and automatically remediate non-compliant S3 buckets that have Block Public Access disabled. This lab will teach you how to create an AWS Config rule and configure automated remediation to automatically revert any unauthorized or accidental changes that disable Block Public Access on S3 buckets. This ensures that buckets remain private by default, improving security posture and preventing unintended public exposure of data.

# Scenario
In this scenario, I’ll implement automated enforcement of S3 Block Public Access settings across S3 buckets within an AWS account. This automation ensures that if Block Public Access is disabled on any bucket—whether through accidental changes or unauthorized actions—AWS Config will automatically detect the non-compliant configuration and restore the secure settings.

<img width="739" height="356" alt="image" src="https://github.com/user-attachments/assets/bac447b1-6bcd-401b-80b7-acbe062ed2de" />




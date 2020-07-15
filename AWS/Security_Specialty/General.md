- [General](#general)
  - [Exam tips](#exam-tips)
  - [Services on Exam](#services-on-exam)

# General
Notes taken from internal security bootcamp

## Exam tips
* Remember, the exam has not been updated for a while. For example, questions on Macie will be about Macie v1.
* Consider the implications of TLS termination on load balancers and how they're implemented. For example, TLS termination and/or passthrough on ALB/NLB and why/when you'd want to do each.
* If you don't have a background in incident response and/or infrastructure security, ensure you spend extra time reviewing these sections
* Tradeoff decisions: cost optimization needs may not always meet performance and/or security best practices
* Emphasis on Infrastructure Security and Data Protection vs. other domains
* IAM policy syntax/definitions/understanding are **very important** to understanding much of the exam
* Know the edge cases
* Don't worry so much about service limits or things that change frequently

## Services on Exam
* Most likely security services to come across:
  * IAM
  * KMS
  * CloudHSM
  * WAF
  * VPC/SG/NACLs
  * CloudWatch
  * Config
* Other services used as examples:
  * S3
  * EC2
  * DynamoDB
  * Lambda
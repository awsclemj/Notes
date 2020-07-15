- [Infrastructure Security](#infrastructure-security)
  - [Exam tips](#exam-tips)
  - [Shared Responsibility Model](#shared-responsibility-model)
  - [S3 Pre-signed URLS](#s3-pre-signed-urls)

# Infrastructure Security
Notes taken from internal bootcamp

## Exam tips
* Subdomains for exam:
  * Edge security
  * Design/implement secure network infra (endpoints, NAT, VPC, peering, SG/NACL)
  * Troubleshoot secure network infrastructure
  * Design/implement host-based security (SSM, proxies, IDS/IPS)
* Good to have hands-on experience in this domain

## Shared Responsibility Model
![AWS Shared Responsibility Model](https://d1.awsstatic.com/security-center/Shared_Responsibility_Model_V2.59d1eccec334b366627e9295b304202faf7b899b.jpg)

## S3 Pre-signed URLS
* `aws s3 presign <object name>`
* User who accesses the resource will have permissions of the enitity that generates the URL
* Default expiry time is 1 hour; can be customized
* You can give a subset of permissions to the accessing user
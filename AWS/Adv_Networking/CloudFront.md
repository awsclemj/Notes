- [CloudFront](#cloudfront)
  - [Overview](#overview)
    - [Private Content](#private-content)
    - [Security Features](#security-features)
    - [Lambda@Edge](#lambdaedge)

# CloudFront
Notes are taken from Linux Academy's Advanced Networking Specialty course

## Overview
* CloudFront is a CDN that can be used to deliver content to users. The content is placed in edge locations across the globe so clients can access it with low latency.
* CloudFront supports HTTP and WebSocket procotols. RTMP distributions are also available for streaming services.
  * When using RTMP, admins but distribute both the media players and the media files
  * Adobe Media Server/Adobe RTMP
* **Origin servers** house source content and can be **EC2 instances, S3 buckets, media services, or even non-AWS origins**
* Origin data is copied to the web distribution. Cache-control headers set by the origin server are used to control how long the data is cached in CloudFront
* You can also control cache by setting object TTLs via the CloudFront distribution
* Objects can be invalidated to be removed immediately

### Private Content
* You can restrict what access a user has to CF content
  * Ex. only authorized users can access certain content vs. guest users
* **Signed URLs**: valid only for a specific time and optionally restricted to certain IP addresses
* **Signed Cookies**: requires auth via public/private key pair
* **Origin Access Identity (OAI)**: restrict access to an S3 bucket to a special CF server associated with the distro
  * Ensures CF is not bypassed for direct S3 access

### Security Features
* **HTTPS**: CF can require HTTPS for all requests. Pulls from origins can also use HTTPS (with the exception of S3)
  * Origin protocol policy can be set to HTTP only, HTTPS only, or Match Viewer.
* **AWS Certificate Manager**: Provisioning and deployment of TLS certs
* **Access Logs**: Create log files that include detailed info about requests received
* **CloudFront Security**: CF supports custom headers from the origin. You can configure the origin to only allow requests that contain the custom header
* **Field-level Encryption**: Secures user-submitted data by encrypting at the edge. The data is encrypted until handed off to your application (requires private key to decrypt)
* **WAF and Shield**: CF supports AWS WAF and Shield, protecting against common web app vulnerabilities and DDoS attacks. WAF can also be used for things like georestriction and IP whitelisting

### Lambda@Edge
* Customize content delivered via CF; Lambda functions are executed at the edge location
* Examples:
  * Inspect cookies/rewrite URLs for A/B app testing
  * Return objects based on device type
  * Return objects based on cookies
  * Generate HTTP responses based on origin requests
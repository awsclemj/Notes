- [Incident and Event Response](#incident-and-event-response)
  - [GuardDuty](#guardduty)
  - [Inspector](#inspector)
  - [Kinesis](#kinesis)
    - [Data Analytics](#data-analytics)
    - [Data Firehose](#data-firehose)
    - [Data Streams](#data-streams)
    - [Video Streams](#video-streams)

# Incident and Event Response
Notes taken from ACG DevOps Pro 2020 course.

## GuardDuty
* A threat detection service that continuously monitors for malicious/unauthorized behavior
* Monitors for:
  * Unusual API calls
  * Unauthorized deployments
  * Compromised instances
* It uses:
  * Amazon threat intelligence feeds
  * Machine learning
  * CloudWatch Events
  * VPC flow logs

## Inspector
* An automated service that assesses your applications for vulnerabilities and produces a security findings report
* Both network assessments (agentless) as well as host assessments (agent required) are supported
  * Network - checks network configuration for ports accessible from outside VPCs
  * Host - check for CVEs and security configuration issues/recommendations
* Benefits:
  * Helps secure your account by identifying security issues
  * It's API-driven, so it's easy to implement in existing DevOps processes
  * Helps reduce risk by alerting on security issues before they become a problem
  * Leverages expertise of security experts
  * Define and enforce your own security standards for your applications

## Kinesis
* Collect, process, and analyze video and data streams in real time
* Four different services:

### Data Analytics 
* Analyze streaming data
* Respond in real time
* Query in SQL

### Data Firehose
* Deliver streaming data to another destination
* No application to manage; just configure the producer (such as a web server)
* Data can be transformed before delivery
* Destinations - S3, Redshift, ElasticSearch, and Splunk
* Accepts record chunks of up to 1000 KB

### Data Streams
* Collect streaming data
* Massively scalable 
* Capture GBs of data per second
* Data available in milliseconds
* Durable; data streamed across three AZs in a region and stored for 7 days

### Video Streams
* Collect streaming video
* Can handle ingestion from millions of devices
* Enables live and on-demand playback
* Take advantage of Amazon Rekognition Video and machine learning frameworks for video
* Access data through APIs
* Build real-time video apps
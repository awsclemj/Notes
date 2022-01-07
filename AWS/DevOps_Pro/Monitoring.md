- [Logging and Monitoring](#logging-and-monitoring)
  - [CloudWatch](#cloudwatch)
    - [Data Retention](#data-retention)
    - [Custom Metrics](#custom-metrics)
  - [EventBridge](#eventbridge)
  - [CloudWatch Logs](#cloudwatch-logs)
  - [X-Ray](#x-ray)

# Logging and Monitoring
Notes taken from ACG DevOps Pro 2020 course.

## CloudWatch
* CloudWatch is used for monitoring your resources in AWS. It has several features:
  * Metric gathering
  * Monitoring and alerting
  * Graphing service
  * Logging service

### Data Retention
* Data points < 60 seconds available for 3-hours (high-resolution)
* Data points ~= 60 seconds available for 15 days
* Data points ~= 300 seconds available for 63 days
* Data points ~= 3600 seconds are available for 445 days (15 months)
* Data points for smaller intervals are aggregated into averages after the retention period has lapsed

### Custom Metrics
* You can publish custom metric data from your application, system, etc. to CloudWatch for monitoring purposes
* Simple API call to put data at interval of your choice

## EventBridge
* AKA CloudWatch Events
* **Event** - a change in your AWS environment
* **Target** - something that will process the event (Lambda, Step Function, etc.)
* **Rules** - matches incoming events and routes them to the target

## CloudWatch Logs
* AWS-managed log aggregation service
* Many AWS-managed services such as Lambda, ECS, CodeBuild, etc. will log to CloudWatch Logs natively
* Log insights allows you to perform advanced queries and visualize your log data
* CloudWatch Agent can be installed on EC2 instances to log system and application log streams

## X-Ray
* Collect data about requests your application serves
* Provides tools you can use to view, filter, and gain insights into that data
* Can help identify issues and optimization opportunities
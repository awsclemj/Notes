## Performance Efficiency Pillar
* Ability to use computing resources efficiently to meet system requirements, and to maintain that efficiency as demand changes and technolgies evolve

### Design Principles
* Traditional environments:
    * Use of the same tech for everything (such as relational databases when NoSQL might be a better fit)
    * Only local resources - global too expensive
    * Use of lots of servers and people to manage the servers
    * Hard to experiment
* In the cloud:
    * Democratize advanced technologies
    * Go global in minutes
    * Use serverless architectures
    * Experiment more often
    * Mechanical sympathy

### Key Services
* Major key service: CloudWatch
* Selection
    * EBS - range of storage options for instances
    * Autoscaling - meet demand
    * S3 - serverless content delivery
    * RDS - wide range of database features/engines
    * DynamoDB - serverless NoSQL/millisecond latency
    * VPC
    * Route 53 - choices for type of DNS routing
    * DX - reduce network latency/jitter
* Review
    * AWS Blog/What's New
* Monitoring
    * CloudWatch - metrics, alarms, notifications
    * Lambda - trigger actions via code
* Tradeoffs
    * CloudFront - CDN/edge caching
    * Elasticache - improve performance of queries
    * SnowBall - quickly ingest data into S3
    * RDS - allows you to scale read-hevy workloads with read replicas

### Best Practices
* Selection
    * Select appropriate resource types (ex. instance size/storage IOPS)
    * Benchmark/load test
    * Monitor performance
* Review
    * Evolve over time to take advantage of new features
    * Load testing - provision load test environments at production scale to test new features/components before merging them to prod
* Monitoring
    * Use CloudWatch to monitor and send notification alarms when thresholds are breached
    * Use automation to work around performance issues by triggering actions through CloudWatch, Kinesis, SQS, or Lambda
* Trade-offs
    * Positioning/caching resources closer to end users (different AWS regions/CloudFront)
    * Use global network of AWS for higher throughput and lower latency
    * Use AWS Direct Connect for predictable latency
    * Use Elasticache or read replicas to improve database performance
    * Test new approaches and capabilities
    * Monitor performance
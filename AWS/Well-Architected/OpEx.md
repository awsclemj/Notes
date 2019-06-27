## Operational Excellence Pillar
* Ability to run and monitor systems to deliver business value
* Continually improve supporting processes and procedures

### Design Principles
* In a traditional environment:
    * Manual changes
    * Tech metrics rather than business outcomes
    * Batch changes since change can be risky
    * Rarely run game days
    * No time to learn from mistakes

* In the cloud:
    * Perform operations with code
    * Annotated docs
    * Frequent, small, reversible changes
    * Anticipate failure
    * Learn from operational failures

### Key Services
* Most key: CloudFormation (Infrastrucure as Code) - full documentation of environment
* Prepare
    * AWS Config and Config rules - compliance and standards for workloads
* Operate
    * CloudWatch allows you to gather operational metrics and logs about your workload
* Evolve
    * AWS Elasticsearch allows you to quickly gather insights about your logs
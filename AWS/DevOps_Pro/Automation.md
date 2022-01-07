- [SDLC Automation](#sdlc-automation)
  - [What is CI/CD?](#what-is-cicd)
    - [Definitions](#definitions)
    - [Benefits](#benefits)
    - [Example Pipeline](#example-pipeline)
  - [CodeCommit](#codecommit)
  - [CodeBuild](#codebuild)
  - [CodeDeploy](#codedeploy)
  - [CodePipeline](#codepipeline)
  - [Testing](#testing)
    - [Why do we test?](#why-do-we-test)
    - [Automated testing in CI/CD](#automated-testing-in-cicd)
      - [Common Types of Automated Tests](#common-types-of-automated-tests)
  - [Artifacts](#artifacts)
  - [Deployment Strategies](#deployment-strategies)
    - [Single target deployment](#single-target-deployment)
    - [All-at-Once Deployment](#all-at-once-deployment)
    - [Minimum in-service deployment](#minimum-in-service-deployment)
    - [Rolling deployment](#rolling-deployment)
    - [Blue/Green Deployment](#bluegreen-deployment)
    - [Canary Deployment](#canary-deployment)

# SDLC Automation
Notes taken from ACG DevOps Pro 2020 course.

## What is CI/CD?

### Definitions
* **Continuous** - forming a series with no exceptions or reversals
* **Integration** - combine one thing with another to form a whole
* **Deployment** - the action of bringing resources into effective action

### Benefits
* Build/test faster
* Decrease code review time
* Automated
* Faster fault isolation
* Additional deployment features (e.g. blue/green deployment)

### Example Pipeline
Source stage (CodeCommit) --> Dev deploy stage (CodeDeploy --> EC2) --> Prod deploy stage (CodeDeploy --> EC2)

## CodeCommit
* Git-based version control repository; AWS-managed and serverless
* Ties in with IAM for authN/authZ

## CodeBuild
* Fully-managed build service. Compiles your code, runs unit tests, produces artifacts, and eliminates need for build servers
* Provides pre-packed build environments as well as support for customized environments
* Scales automatically to meet build requirements
* Uses `buildspec.yml` file to define build steps

## CodeDeploy
* Managed deployment service that automates deployments to EC2, on-prem VMs, and Lambda functions
* Makes it easier to:
  * Rapidly deploy new features
  * Update Lambda function versions
  * Avoid downtime during deployment
  * Handle complex deployment processes without human intervention
* Uses `appspec.yml` file to define deployment steps

## CodePipeline
* Fully-managed and fully-automated CD tool
* Serverless and easy to configure; pre-built plugins, but you can also roll your own
* Create, configure, and modify all stages of your software release process
* Implement testing and customize the entire deployment process

## Testing

### Why do we test? 
* Meet requirements defined
* Ensure code performs efficiently
* Usability checks
* Ensure it responds correctly to all kinds of inputs

### Automated testing in CI/CD
* Automated execution of tests
* Actual outputs are compared against expected outputs
* Fast, continuous feedback; immediately notified of failures. Build halted on failures. 

#### Common Types of Automated Tests
* **Unit** - test individual units/functions in code to determine if they're operating correctly

## Artifacts
* A product or by-product produced during the SDLC, e.g.:
  * Compiled binary
  * Container images
  * Source code
  * Documentation
  * Use cases
  * Class diagrams

## Deployment Strategies

### Single target deployment
* Build code and deploy it to a single server
* Used for small development projects, especially for legacy, non-HA infrastructure
* New application version is installed on server
* Brief outage during installation. No secondary servers, so testing is limited. Rolling back requires deletion and re-installation of previous app version

### All-at-Once Deployment
* Deployment happens in once step, but for multiple targets
* More complicated than single target due to often needing orchestration tooling 
* Shares negative aspects of single target in that limited ability to test, deployment outages, and less than ideal rollback

### Minimum in-service deployment
* Deployment happens in multiple stages
* A few moving parts; orchestration and health checks are required
* Deploys new versions of code to as many targets as possible while keeping a defined minimum number of servers in service
* After new code is deployed to other servers and passing health checks, the servers kept in service are then updated
* Allows for automated testing; deployment targets are assessed/tested prior to completion
* Generally no downtime and often quicker than a rolling deployment 

### Rolling deployment 
* Deployment happens in multiple stages. Number of targets per stage is user-defined
* A few moving parts; orchestration and health checks are required
* Overall health isn't necessarily maintained; health checks are generally only applied to the targets being updated in a rollout stage
* Can be least efficient due to time taken
* Allows for automated testing; deployment targets are assessed/tested prior to completion
* Generally no downtime, assuming number of targets per run doesn't impact the application
* Can be paused, which allows limited multi-version testing 

### Blue/Green Deployment
* Requires advanced orchestration tooling; deployment of new environment and new application code are automated
* Carries cost consideration -- 2 separate environments must be maintained for the duration of the deployment 
* Rapid deployment process -- entire environment (blue or green) is deployed all at once
* Cutover and rollback is clean and controlled (simple DNS change)
* Health/performance of green environment can be tested prior to cutover
* Using advanced template systems (CloudFormation), the entire process can be automated

### Canary Deployment
* Similar to blue/green -- new environment is spun up, but the DNS is pointed to both environments with weights
* Slowly increase weight to green environment (Route 53 weighted round robin)
* This is the same method used for A/B testing
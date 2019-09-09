## AWS Developer Tools
* Notes taken from Linux Academy DevAssociate course.

### X-Ray
* Traces requests as they move through your application; collects data you can view, filter, and sort.
* Concepts:
    * **Segments** - Data about the work done by your application (request/response, subsegments, issues)
    * **Subsegments** - more granular view inside segments
    * **Service Graph** - JSON doc containing info about how app service/resources interact
    * **Traces** - Trace IDs track requests as they go through your app:
        * X-Ray supported service adds HTTP trace ID header, which propagates downstream to track requests
    * **Sampling** - X-Ray applies and algorithm to sample request data but the frequency is configurable 
    * **Tracing Header** - the header applied to the HTTP requests. Ex.: `X-Amzn-Trace-Id: Root=1-5759e988-xxxxyyyyzzzz;SampleId=1`
    * **Filter Expressions** - Used to search through your traces by different characteristics
    * **Annotations and Metadata** - additional way to store searchable (annotations) and non-searchable (metadata) about traces
    * **Errors, Faults, Exceptions** - Application errors are recorded and categorized:
        * Error - 4xx errors (client)
        * Fault -  5xx errors (server)
        * Throttle - 429 errors (Too many requests)

### CodeCommit
* Managed private git repository
* Benefits:
    * Stored encrypted in AWS
    * No limits on file types or size of repo
    * Migrate from other repositories
    * Use familiar git workflows

### CodeBuild
* Managed build service that can compile code, run unit tests, and produce deployment artifacts
* Benefits:
    * Fully managed, on-demand, and preconfigured environments for many programming languages
* Concepts:
    * **Build Project** - defines how CodeBuild runs a build
        * Where is source code, what build env to use, what commands to run, where to store output?
    * **Build Environment** - OS, language runtime, and tools for CodeBuild to use
    * **Build Spec** - YAML file that describes collection of commands/settings for CodeBuild to run

### CodeDeploy
* Automates deployments of your applications to EC2, Lambda, and on-prem environments
* Benefits:
    * Minimize downtime, stop/rollback deployments, centralized control
* Deployment Types:
    * **In-place** - existing servers are updated with the new version of and application
    * **Blue/green deployments**:
        * EC2 - new instances with newly deployed application are launched 
            * ELB routes traffic to the new deployment. Old deployment is kept for some time in case rollback needed
        * Lambda - a new function version is created and traffic can be split between the old and new version
            * ***Canary*** - percentage of traffic shifted to new version, CodeDeploy waits for a specified time and fully shifts if no errors
            * ***Linear*** - Traffic shifted in equal increments with equal number of minutes between each
            * ***All at once*** - Traffic is completely shifted to the new version of the Lambda function
* Hooks
    * AppSpec file:
        * Lambda - which functions to deploy and which functions to use as validation tests
        * EC2/On-prem - What to install (from source repo) and what lifecycle event hooks to run
    * Lifecycle Hooks:
        * Available hooks depend on deployment type
        * Allow arbitrary scripts to be run during deployment
        * Examples: BeforeInstall, AfterInstall, ApplicationStart, ApplicationStop, ValidateService

### CodePipeline
* Continuous delivery service that can model, visualize, and automate software release process
* Benefits:
    * Automatic, consistent release process
    * Real-time status
    * Integrations for:
        * Source control - S3, CodeCommit, GitHub
        * Build - CodeBuild, CloudBees, Jenkins, SolanoCI, TeamCity
        * Test - CodeBuild, HOE StormRunning Load, BlazeMeter, Nouvola, Runscope, Ghost Inspector
        * Deploy - CloudFormation, CodeDeploy, ECS, Beanstalk, OpsWorks Stacks, XebiaLabs
* Concepts:
    * **Pipelines** - workflow that describes your software release process
    * **Artifacts** - files/changes that will be worked on by the actions and stages in the pipeline
    * **Stages** - pipelines are broken into stages
        * Build stage - code is built and tests are run
        * Deployment stage - updates deployed to environments
    * **Actions** - stages contain at least one action which take an artifact as input, output, or both
    * **Transitions** - progressing from one stage to another inside the pipeline

### CodeStar
* Used for creating, managing, and working with AWS projects
* Benefits:
    * Project templates for various programming languages
    * Helps manage user access/IAM
    * Integration with some IDEs
    * Visualize, collaborate on projects in a central place
        * Metrics such as CPU util
        * JIRA issues
        * Commit history/changes
        * Continuous deployment details

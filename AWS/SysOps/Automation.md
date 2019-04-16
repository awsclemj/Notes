## Automation

* **CloudFormation**
	* What is it?
		* Allows you to manage, configure, and provision AWS infra as code
		* Resources are defined using CloudFormation template
		* CFN interprets the template and makes the appropriate API calls to create resources
		* Supports YAML or JSON 
	* Benefits
		* Infra provisioned consistently
		* Less time and effort that configuring manually
		* Version control and peer review templates
		* Free to use (only pay for what you create)
		* Manage updates/dependencies
		* Rollback/delete entire stack
	* Templates
		* Use YAML or JSON to describe the end state of the infra you are provisioning or changing
		* After creating the template, you upload it to CFN using S3
		* CFN reads the template and makes the API calls on your behalf
		* Resulting resources are called  stack
	* Template sections:
		* Description
		* Metadata
		* Parameters
			* Custom input values
		* Conditions
			* e.g. provision resources based on environment
		* Mappings
			* User-defined values (e.g. an AMI to use in a region)
		* Transform
			* Can be used to include snippets of code located in S3
		* Resources
			* AWS resources you are deploying
			* Most important section
		* Outputs
			* Outputs will be displayed on the console
		* Resources is the only mandatory section of a template

* **Elastic Beanstalk**
	* What is it?
		* Service for deploying a scaling web apps developed in many popular languages
			* Java
			* .NET
			* PHP
			* Node.js
			* Python
			* Ruby
			* Go
			* Docker
		* Can be deployed onto widely used application server platforms such as
			* Apache Tomcat
			* nginx
			* Passenger
			* IIS
		* Users can focus on writing code and don't need to worry about underlying infra
		* You upload the code and Beanstalk will handle deployment, capacity, load balancing, autoscaling, and application health
		* Retain full control of the underlying AWS resources powering your application and you pay only for the resources required
	* Benefits:
		* Fast and simple way to deploy an application on AWS
		* Automatically scales up/down
		* Select EC2 instance type that suits your application
		* Retain full administrative control over the resources or have Beanstalk do it for you
		* Managed platform updates
		* Monitor and manage application health via dashboard

* **AWS OpsWorks**
	* Allows you to automate your server config using Puppet or Chef
		* Uses managed Puppet/Chef instances
	* Enables configuration management for your OS and applications
	* Allows you to automate server config using code


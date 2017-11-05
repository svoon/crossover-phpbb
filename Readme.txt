Readme Document
Create a txt file with the following information
1. Instructions to evaluate the work done (e.g. AWS account credentials) and explain the deliverables
2. Assumptions you have made - it is good to explain your thought process and the assumptions you
have made
3. Requirements that you have not covered in your submission, if any
4. Issues you have faced while completing the assignment, if any
5. Constructive feedback for improving the assignment

Instructions to evaluate the work
  - First of all I have to admin the solution doesn't work. Although I've used working (tested) reference AWS architecture, the last web stack doesn't work. Unfortunately due to time constraints I cannot finish it, therefore submitting in non-working state (I have time only during this weekend to work on this project, cannot spend tomorrow 3rd day on it as I'm at the office till late night).
  - S3 bucket contains all the source codes and all other files required to run this
  - I was trying to prepare highly available solution with scaling possibilities and fulfilling all the requirements and assumptions. 
  - For me is not very clear what should be part of the readme.txt document, most probably some of the points I would rather insert into Design.doc. As I would like to avoid having same information in two documents, it is either here, in Design.doc or in Deployment.doc


Assumptions
  - VPN/DX - as there is no requirement to setup connectivity to Customer's onpremise network, neither VPN or DX is used. In case of such requirement both options should be considered
  - Only publicly available regions are offered (e.g. China region which requires business entity to be registered within China country is not offered). Same for Gov region.
  - Application should be deployed in High Available mode to minimize downtime, and any single point of failure should be removed. Disaster recovery is not listed in requirements, therefore is for lowering the costs kept out of scope.
  - SLA for the application is not higher than 99.95% - this is the SLA for multi-AZ RDS deployment
  - Backup is required, but it is not requested to perform off-site backup - into on-premise datacenter or into different AWS region
  - Workload (number of users) is currently unknown, and there are not available any performance data, but solution should be prepared for higher load in future, even now starting minimal.
  - Stable version is required
  - IPv6 traffic is not required


Considerations
  - Before diving into high/low level design of AWS infrastructure, first should be considered AWS's reference architecture (https://aws.amazon.com/architecture/) , to avoid duplicating work and leverage of the vendor recommendation and best practices.
  - The most similar application available in the reference architecture is WordPress
	• Web/App tier are by default combined
	• MySQL is used as backend
	• Wordpress supports memcache backend, I was not able to find this is supported in MyBB (*see below)
  - Cloudformation template is based on the above mentioned WordPress one, with performed required changes to comply with the MyBB installation:
	• Removed Elasticache component as EC is not supported in MyBB
	• Application installation script was reworked to support MyBB
  - RDS Audora is used as a MySQL backend replacement, as it provides better performance, automated failover and has better failover scenarios (https://aws.amazon.com/rds/aurora/faqs/)
  - The total amount of expected time is planned to 10 hours. Within the 10 hours we need to plan time to cover all the required areas, rather to deepdive into one area and not deliver the final product. Into consideration has to be taken all the document writing, code scripting and as well testing. Phased approach is proposed to deliver within the "ordered" amount of time viable product, and add additional functionality later based on requirements and its priorities.
  
	  
  - Deployment consideration
	○ Unfortunately official sources of MyBB application doesn't provide unattended installation. Unattended installation is needed for autoscaling mechanism, where the application has to be deployed automatically without human intervention.
	○ Therefore as a first step I've downloaded and installed MyBB manually on my dev server to understand the deployment process:
	  § There are few bash commands in documentation available https://docs.mybb.com/1.8/install/#ssh-quick-install which prepares the basic installation
	  § Unfortunatelly during manual web installation are than generated files inc/settings.php and inc/config.php , which are empty after MyBB source download
	  § It will be required to store these files within my Git project / S3 bucket and generalize the variables which are given by user during the manual installation to be able to be replaced
	  § During this investigation I've found few projects on GitHub, who were already solving unattended MyBB installation, so to don't start from ground zero, I've integrated some scripts from project https://github.com/sahilsehgal81/cloudformation-crossover-mybb.git
	○ Original assumption MyBB doesn't support memcached was identified during the deployment as incorrect, as the configuration files provides possibility to use memcached. Adding this feature into Improvements section
	○ 
	
Improvements
There are additional areas for improvement which are currently not covered. I would proposed phased approach to bring the environment in small steps to the desired state. Some of the areas will depend on the customer's requirements:
  1) Remove single point of failure - configure RDS as highly available
  2) Add CloudWatch Alarms for:
	a. Financial spent
	b. ELB latency
	c. DB instance failures
  3) Define network ACL
  4) Use bastion server for managing infrastructure with configurable parameter for SSH source Ips access to bastion server
  5) HTTPS support
	a. All the HTTP traffic should be encrypted with SSL. In case customer doesn't have yet certificate signed by some of the authority, he might use LetsEncrypt SSL certificate, which comes for free
  6) Shared storage to be used
  - Current application implementation currently doesn't support S3 storage
  - Patching application to support S3 is not a good option, as it will prevent (or complicate) applying updates/bugfixes in future
  - In case customer will decide to invest into S3 integration, the code should be provided into upstream, which will than enable future upgrades
  - As 
  7) CloudFront - use CloudFront for caching static data and Web Application Firewall. This will allow:
  - Lower the costs (CloudFront data traffic is cheaper than from S3/EC2)
  - Increase the web page responsiveness, as the static data are closer to the end user 
  Source: https://docs.mybb.com/1.8/faq/using-a-cdn-with-mybb/
  8) Backup application data
  9) Design, Automate and regularly test the backup/recovery scenarios. 
  10) Migrate the deployment into AWS CodeDeploy 
  11) Extend deployment for Memcached  - In AWS it will be AWS ElastiCache, which is in memory datastore compatible with Memcached, which is coming as managed service.
  12) IPv6 support

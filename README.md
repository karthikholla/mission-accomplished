mission-accomplished
====================

Question 1
----------

* Backend servers
  * Horizontal and vertical scaling.
  * Auto Scaling Groups in AWS. Horizontal scaling common approach when running on the cloud, leveraging it commodity hardware. It is the most preferred way of scaling.
  * You could define the minimum and maximum size to scale in Auto Scaling Groups and assign a scaling policy and metric to trigger the elasticity. Some of the sample metrics are CPU, Queue length, Network IO.
  * You can use Application LB or Network LB on the path redirect.

* Database performance
  * Unlike EC2 resources, I believe database are a static entity and needs to be properly configured in the first place. The main requirement for any DB is read & write IOPS.
  * It must be multi AZ with SSD volumes for high IOPS. Having multiAZ deployment, the HA is covered.
  * [Vacuum][] needs to be done on the database regurlarly.
  * Apply TTL if the client application is caching the Domain Name Service (DNS) data using read replicas to horizontally scale your database.
  * Read replica in a different AWS Region closer to your users for better performance

* Message Bus performance(RabbitMQ)
  * One way to have load balancer to connect to many nodes. Having this when multiple nodes are attached to increase the throughput.
  * Load balance by adjusting the distribution of publishers and consumers.
  * Exchange-to-exchange bindings improve decoupling, increase topology flexibility
  * More Worker Queues help parallelize and distribute workloads and  Distributing worker queues in the cluster helps scale.

* Docker + Kubernetes
  * Docker-swarm
  * We don't use k8s at production grade.

Question 2
----------

* User Passwords(actual clients) & SysAdmin passwords
  * I have use-case solution based on the above question. I have implemented [Passbolt][] for the above problem. We have self-hosted passbolt setup in our organisation. However, for better support and enhancement, we can use cloud based hosted solution. It is web based and easy to colloboration
  * For shorter teams, passbolt_docker which can used store and share between individuals in the team. https://help.passbolt.com/hosting/install/ce/docker.html

  ![Passbolt](https://github.com/karthikholla/mission-accomplished/blob/master/images/passbolt.png)

* Passwords for connections to DBs, which are used by backend services
  * Database URL credentials as stored in the jenkins ENV variable., which is replaced into the actual connection string in the jenkinsfile.
  Eg:
  `withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'Application-XYZ' ,usernameVariable: 'env_1', passwordVariable: 'env_2']]) {
   sh 'sed -i "s&<% ENV_1 %>&$env_1&g" .build/env'
   sh 'sed -i "s&<% ENV_2 %>&$env_2&g" .build/env'
 }`

* Docker file configurations that should include passwords
  * We make it as ENV in the Dockerfile or docker-compose file and  replace the env key with value in the jenkins file. In the jenkins server, we store the credentials. Also, we use `docker secret` command to store secrets.

* SSL/Encryption Keys
  * We use AWS Certificate manager for SSL and AWS KMS to encrypt all the necessary AWS resources.

Question 3
----------

* Postgres SQL
  * Grant users the minimum access they require to do their work, nothing more.
  * Isolate the database port from other network traffic.
  * Set appropriate monitoring and logging for database queries.
  * Restrict access to configuration files (postgresql.conf and pg_hba.conf)
  * Keep backups, and have a tested recovery plan.
  * AAA model.
  * Have AWS WAF to avoid sql injections threats.

* Docker + Kubernetes
  * Docker containers with the -u flag so that they run as an ordinary user instead of root.
  * Configure [Docker-control-groups][]
  * Use namespaces in Docker to isolate containers.
  * Avoid use of repos you don’t trust. In particular, avoid public repos.
  * command `sudo export DOCKER_CONTENT_TRUST=1` warns when you attempt to pull down an image that isn't signed, Docker will inform you.

* General infrastructure & Data
  * Confined VPC
  * Firewalls in the form of security Groups
  * Applications are deployed on hardened servers. Also, Patching the ec2 machines regularly.
  * Monitor system for any changes or intrusion.
  Ref: [AWS_Security_Best_Practices][]

Question 4
----------

* Frontend
* BackEnd
* DBs
* Message Bus

We use Cloudwatch logs instead of logstash as a place holder for the logs. Installed cloudwatch agent on the host machine to push the all system logs to the Cloudwatch. We use **awslogs docker logging driver** to push the application logs from the docker containers.
* Application Logs (**Frontend, BackEnd, Message Bus**)
  * Some application use filbeat env variable in the application to push all the application logs is directly ingested to AWS ES. Few doceker applications use **awslogs docker logging driver**, push the logs to the Cloudwatch group and then push it to ES.
* System Logs
  * Install Cloudwatch logs agents & use it to push system logs - syslog, ossec.log and other necessary system logs.
![Image of Architecture](https://github.com/karthikholla/mission-accomplished/blob/master/images/diagram2.png)
* DB
  * Postgres RDS and redshift logs are monitored through cloudwatch.

Question 6
----------

All new developers are instructed to upload their pubkey to the github repo wrt their team. All developers pubkey are stored in github as a code. During the infrastructure creation ie., ec2 provisioning, the depending on the particular project team’s pubkey are replaced in the script below to provide the ssh access.

* Shell script to granting ssh access to servers

  `useradd developer;usermod -s /bin/bash developer;mkdir -p /home/developer/.ssh;echo "<pub_key_here>" > /home/developer/.ssh/authorized_keys;chmod 600 /home/developer/.ssh/authorized_keys;chmod 700 /home/developer/.ssh;chown -R developer:developer /home/developer;`

Since we are moving towards immutable infrastructure, there is no need for revoking access manually. We just remove the developer's pubkey from the group, so that during the ec2 creation automatically the developer lose SSH access. However, we can remove

  `deluser --remove-home developer`

Optionally you can use `--remove-all-files` to remove all the files owned by that user. We could setup the Jenkins job for automating the deletion process from all the VM's.

![SSHflow](https://github.com/karthikholla/mission-accomplished/blob/master/images/ssh.png)

[Passbolt]: https://www.passbolt.com/
[Vacuum]: https://github.com/awsdocs/amazon-rds-user-guide/blob/master/doc_source/CHAP_BestPractices.md#working-with-the-postgresql-autovacuum-feature
[Docker-control-groups]: https://docs.docker.com/engine/security/security/
[AWS_Security_Best_Practices]: https://d1.awsstatic.com/whitepapers/Security/AWS_Security_Best_Practices.pdf

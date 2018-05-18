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
  * <pending>

* Docker + Kubernetes
  * <pending>

Question 2
----------

* User Passwords(actual clients) & SysAdmin passwords
  * I have use-case solution based on the above question. I have implemented [Passbolt][] for the above problem. We have self-hosted passbolt setup in our organisation. However, for better support and enhancement, we can use cloud based hosted solution. It is web based and easy to colloboration
  * For shorter teams, passbolt_docker which can used store and share between individuals in the team. https://help.passbolt.com/hosting/install/ce/docker.html

  ![Passbolt](https://github.com/karthikholla/mission-accomplished/blob/master/images/passbolt.png)

* Passwords for connections to DBs, which are used by backend services
  * <pending>

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

* General infrastructure
  * <pending>

* Data
  * <pending>


Question 4
----------

* Frontend
* BackEnd
* DBs
* Message Bus

To Accomplish this we are using Cloudwatch logs instead of logstash. Installed cloudwatch agent on the host machine to push the all system logs to the CW and using awslogs docker logging driver to push the application logs.

* Application Logs
  * Using filbeat env variable, all the application logs is directly ingested to AWS ES. Using awslogs logging driver, push the logs to the CW group and then push it to ES.

* System Logs
  * Install CW logs agents & use it to push system logs. successfully pushed the syslog & ossec.log to ES via filebeat docker, But, by pushing the system logs directly to ES directly, the filters/grok cannot be imposed on the logs.

![Image of Architecture](https://github.com/karthikholla/mission-accomplished/blob/master/images/diagram2.png)


Question 6
----------

All developes pubkey are stored in github as a code. During the infra creation the depending on the project particular team’s pubkey are replaced in the script to provide the ssh access.

* Shell script to granting ssh access to servers

  `useradd developer;usermod -s /bin/bash developer;mkdir -p /home/developer/.ssh;echo "<pub_key_here>" > /home/developer/.ssh/authorized_keys;chmod 600 /home/developer/.ssh/authorized_keys;chmod 700 /home/developer/.ssh;chown -R developer:developer /home/developer;`

Since we are moving towards immutable infrastructure, there is no need for revoking access. However, we can remove

  `deluser --remove-home developer`

Optionally you can remove `--remove-all-files` to remove all the files owned by that user,

[Passbolt]: https://www.passbolt.com/
[Vacuum]: https://github.com/awsdocs/amazon-rds-user-guide/blob/master/doc_source/CHAP_BestPractices.md#working-with-the-postgresql-autovacuum-feature
[Docker-control-groups]: https://docs.docker.com/engine/security/security/

### Application Load Balancer

An Application Load Balancer (ALB) is a service provided by Amazon Web Services (AWS) that distributes incoming application traffic across multiple targets, such as EC2 instances, containers, and IP addresses, within an Amazon Virtual Private Cloud (VPC). ALB operates at the application layer (Layer 7) of the OSI model and offers advanced routing and load balancing capabilities.

Two common deployment techniques used in software development and release management are canary deployment and blue-green deployment. By gradually implementing changes, they both seek to reduce the risk connected with releasing new versions of an application or service.

#### Canary Deployment
---
Canary deployment involves releasing a new version of an application or service to a small subset of users or servers before making it available to the entire user base. The process is similar to how a canary in a coal mine was used to detect potential hazards. The idea is to test the new version in a controlled environment to identify any issues or performance problems before exposing it to a wider audience.

Key steps in a canary deployment include:

* Create a separate production environment or use feature flags to control access to the new version.
* Direct a small percentage of the user traffic to the new version while the majority still uses the stable version.
* Monitor the performance, error rates, and other metrics of the canary group closely to ensure it behaves as expected.
* Gradually increase the percentage of traffic routed to the canary until it represents the entire user base.
* If any issues arise during the canary phase, roll back to the stable version quickly to minimize the impact on users.

Canary deployments enable quick validation and risk reduction by identifying potential issues early. They are especially helpful for deploying important modifications or mission-critical programmes.

#### Blue green deployment
---
Blue-green deployment includes keeping two identical environments, referred to as "blue" and "green." The new version of the application is represented by the green environment, while the stable version is represented by the blue environment.

Key steps in a blue-green deployment include:

* The blue environment serves as the primary production environment, handling all user traffic.
* Deploy the new version of the application to the green environment, ensuring it is tested and validated.
* Switch the router or load balancer configuration to redirect traffic from the blue environment to the green environment.
* Monitor the performance and behavior of the green environment, allowing time to identify any issues.
* If any problems are detected, easily roll back to the blue environment by updating the router or load balancer configuration.
* Once confident in the green environment, it becomes the new production environment, and the blue environment can be used for future updates.

Because the old environment is still in place and can be rapidly restored in the event of problems, blue-green deployments provide a dependable and reversible option to distribute new versions. This strategy reduces downtime and offers a simple rollback method.

Both the blue-green deployment and the canary deployment strategies have their benefits and are appropriate in certain situations. The decision between them is based on your application's particular requirements, your level of risk tolerance, and the preferred deployment procedure.

In order to demonstrate these, I need 3 versions of the applications. Here I'm writing a simple small index pages with 3 versions like version-1, version-2 and version-3. To make those versions please follow the below steps.

#### Step 1
---

* Create a launch configuration

Here I'm assigning the name for launch configuration as shopping-app-version-1 with Amazon AMI, it's ID is ami-0607784b46cbe5816 with instance type t2.micro.

<img width="693" alt="Screenshot 2023-06-02 105515" src="https://github.com/arshadrebin/ALB/assets/116037443/8f90e579-a602-4f56-812a-c9743d1a45b4">


Under the additional details give the user data as follows.

```
#!/bin/bash


echo "ClientAliveInterval 60" >> /etc/ssh/sshd_config
echo "LANG=en_US.utf-8" >> /etc/environment
echo "LC_ALL=en_US.utf-8" >> /etc/environment
systemctl restart sshd.service

yum install httpd php -y

cat <<EOF > /var/www/html/index.php
<?php
\$output = shell_exec('echo $HOSTNAME');
echo "<h1><center><pre>\$output</pre></center></h1>";
echo "<h1><center>shopping-app-version-1</center></h1>"
?>
EOF
```

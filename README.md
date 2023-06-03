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


systemctl restart php-fpm.service httpd.service
systemctl enable php-fpm.service httpd.service
```

Add the required security group, key pair then create the Launch Configuration.

### Step 2
---

* Create Auto Scaling Group using the above Launch configuration.

* **ASG name** : shopping-app-version-1
* **LC** : shopping-app-version-1

<img width="403" alt="Screenshot 2023-06-02 110123" src="https://github.com/arshadrebin/ALB/assets/116037443/1c330276-11f9-45ba-86d2-07f74f7e0fa5">

Select the VPC and Availability Zones and subnets. Here I'm going with default VPC and AZ ap-south-1a and ap-south-1b.

On the Configure group size and scaling policies, mention Group size as follows.

**Desired capacity** = 2
**Minimum capacity** = 2
**Maximum capacity** = 2

<img width="341" alt="Screenshot 2023-06-02 110508" src="https://github.com/arshadrebin/ALB/assets/116037443/b7060abf-25a5-4c8a-ba1e-d146dafcf000">

At the last step mention a tag name as version-1 to identify the instance created by this ASG.

![242018558-dc9e640a-5963-4ca1-8ff8-13e000c6bd00](https://github.com/arshadrebin/ALB/assets/116037443/1813ed9f-7f85-4ced-a280-ffe1d8f09003)

### Step3
---
* Create a target group

* Choose target type : instances
* Target group name : shopping-app-version-1

Giving the Health check path : /index.php

Leave the rest of the field as it is.

### Step 4
---

* Assign Target Group to Auto Scaling Group.

Go to the Auto Scaling Group >>> Select Auto Scaling Group >>> Actions >>> Edit and select TG as below.

<img width="446" alt="Screenshot 2023-06-02 113312" src="https://github.com/arshadrebin/ALB/assets/116037443/b6e70bc7-1e9e-4bd9-8ac2-b5306f811782">

### Step 5
---

Create Application Load Balancer.

Go to Load Balancer section >>> Select create icon nearby Application Load balamncer section.

<img width="568" alt="Screenshot 2023-06-02 113522" src="https://github.com/arshadrebin/ALB/assets/116037443/88cd4945-3bc6-40a0-96d3-d17d96c94412">

Create Application Load balancer with the below details.

Load balancer name : shopping-app
On the Network mapping select the VPC and AZ's.
Assign the required SG.
Listener : Change it to HTTPS and forward to the shopping-app-version-1 TG
Load the certificate from ACM and create the ALB as follows.
<img width="568" alt="Screenshot 2023-06-02 113905" src="https://github.com/arshadrebin/ALB/assets/116037443/0be8b82a-380e-4082-b1bc-6356a84b274d">


<img width="568" alt="Screenshot 2023-06-02 113927" src="https://github.com/arshadrebin/ALB/assets/116037443/4b091a9b-e6e6-4c33-9095-2b1463131fda">

### Step-6
---

Then we need to edit the listener to redirect the HTTP request to HTTPS. For this, select the above created ALB >> Listener tab >> Add listener as below


<img width="407" alt="Screenshot 2023-06-02 114620" src="https://github.com/arshadrebin/ALB/assets/116037443/861386da-9ccd-4cda-84f0-a74c5c031512">

Again we need to make some changes in the listener section. So select the HTTPS:443 protocol in the listener section and click on the actions >> Manage Rules.
You will get an interface as below.

<img width="760" alt="Screenshot 2023-06-02 114814" src="https://github.com/arshadrebin/ALB/assets/116037443/2786efca-d2ec-463e-8a6c-a808964b1aa9">

On this interface, click on the edit button on the right top corner and then ediot the default rule and add the entry like below.

<img width="657" alt="Screenshot 2023-06-02 115610" src="https://github.com/arshadrebin/ALB/assets/116037443/b3c4e69f-9447-40a5-871b-fa607be41559">

Then add a host header for the domain like as per yours and forward it to the Target Group which we created previously.

### Step 7
---

* Point the domain to ALB via Route53.

Click on Route 53 >> DNS management >> Click on Hosted Zone >> Select the Hosted Zone >> Create Record as follows.

<img width="736" alt="Screenshot 2023-06-02 120444" src="https://github.com/arshadrebin/ALB/assets/116037443/91266c27-d75e-4c01-8ff1-7068387d2c99">

Then you will get the site as below from the 2 instances since we deployed 2 instances in our ASG.

<img width="874" alt="Screenshot 2023-06-02 122554" src="https://github.com/arshadrebin/ALB/assets/116037443/332f1f42-0c12-47a7-991a-de6e50fd5a5c">

<img width="864" alt="Screenshot 2023-06-02 122609" src="https://github.com/arshadrebin/ALB/assets/116037443/7e44b814-b1b9-42bb-801c-4f7834bff2fa">

Now I want to change the version-1 of the application to version-2 with out any down time with 90% traffic to version-1 and 10% traffic to the version-2. In this case we are doing canary deployment. Please note in order to do so the Load Balancer must support canary deployment. ALB is such a one which is capable of it.

![242182662-12f183b5-93ea-43a1-bafa-474d067de42d](https://github.com/arshadrebin/ALB/assets/116037443/75ba95af-968a-417c-9aaf-501a0b99f860)

Repeat the steps from step-1 to step-5 by changing the names to shopping-app-version-2 instead of version-1.

Then go to load balancer >> select the LB >> listener >> select the HTTPS:443 >> Actions >> Manage Rules.

Edit the rules as follows.


<img width="639" alt="Screenshot 2023-06-03 101615" src="https://github.com/arshadrebin/ALB/assets/116037443/643eba9a-2374-4fd2-8f6d-fd4371141904">

If we are applying this rules, the end users get the simultaneous version at the time of new request or refresh.  In order to prevent this, we are selecting group-level-stickiness.

In the context of load balancing, stickiness refers to the ability of a load balancer to maintain a consistent connection between a client and a particular backend server. When stickiness is enabled, the load balancer ensures that subsequent requests from the same client are forwarded to the same backend server, rather than distributing them across multiple servers.

<img width="628" alt="Screenshot 2023-06-03 102325" src="https://github.com/arshadrebin/ALB/assets/116037443/c8da7ee3-b36a-4410-9e6c-2ec50e1acb76">

Here we have applied 1 hour of the stickiness. So a user will get the same version of the application what he/she is getting initially rather than switching it into another.


Then we will gradually change the ratio of the traffic weight and finally make the whole traffic to version-2. This called as canary deployment.

<img width="629" alt="Screenshot 2023-06-03 102603" src="https://github.com/arshadrebin/ALB/assets/116037443/e4c63460-e5e9-43ef-9ada-94c8ddb0b74f">

Now I want to change the version-2 of the application to version-3 with out any down time with 100% traffic to version-3 and 0% traffic to the version-2. In this case we are doing blue green deployment.


![242183579-1432a7ca-8a48-4052-8cd7-6ed36dde92f9](https://github.com/arshadrebin/ALB/assets/116037443/19bf85ed-594b-413f-b0dc-98e5df89b030)

Repeat the steps from step-1 to step-5 by changing the names to shopping-app-version-3 instead of version-1.

Then go to load balancer >> select the LB >> listener >> select the HTTPS:443 >> Actions >> Manage Rules.

Edit the rules as follows

<img width="621" alt="Screenshot 2023-06-03 104458" src="https://github.com/arshadrebin/ALB/assets/116037443/b57fb526-4500-4729-9a01-88c768b249ab">

The above example is called as blue green deployment. The version will completely shift to the version 3 one from previous versions.



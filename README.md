### Application Load Balancer

An Application Load Balancer (ALB) is a service provided by Amazon Web Services (AWS) that distributes incoming application traffic across multiple targets, such as EC2 instances, containers, and IP addresses, within an Amazon Virtual Private Cloud (VPC). ALB operates at the application layer (Layer 7) of the OSI model and offers advanced routing and load balancing capabilities.

Two common deployment techniques used in software development and release management are canary deployment and blue-green deployment. By gradually implementing changes, they both seek to reduce the risk connected with releasing new versions of an application or service.

#### Canary Deployment
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


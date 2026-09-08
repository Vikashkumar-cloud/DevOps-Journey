# DevOps / AWS Interview Questions & Answers

**1. Explain the complete CI/CD pipeline you have implemented using Jenkins, GitHub, and AWS services.**
Interview Answer

“In my project, we implemented an end-to-end CI/CD pipeline using GitHub, Jenkins, Docker, Amazon ECR, and Amazon EKS.

The flow starts when a developer works on a feature branch and raises a Pull Request to the main branch. After the code is reviewed and merged into the main branch, a GitHub webhook triggers Jenkins.

Jenkins then performs the following stages:

Checkout – Jenkins pulls the latest code from GitHub.
Build – We build the application using Maven.
Unit Testing – We execute automated unit tests.
Code Quality – SonarQube performs static code analysis.
Dependency Scan – OWASP Dependency-Check scans application dependencies.
Docker Build – Jenkins builds a Docker image using the Dockerfile.
Image Scan – Trivy scans the Docker image for vulnerabilities.
Push to ECR – If all quality and security checks pass, Jenkins authenticates with AWS and pushes the image to Amazon ECR.
Deployment – Jenkins updates the Kubernetes deployment with the new image tag and deploys it to Amazon EKS.
Verification – We verify pod status, rollout status, application health, and monitoring dashboards using tools such as Prometheus and Grafana.

For AWS authentication, I prefer using IAM roles or short-lived credentials rather than hard-coded AWS access keys.

So, the overall flow is:

Developer → GitHub → Webhook → Jenkins → Build/Test → SonarQube → OWASP → Docker → Trivy → ECR → EKS → Monitoring.”

Strong closing line

“The main objective was to automate the complete software delivery lifecycle while introducing quality gates, security scanning, and controlled production deployment.”

**2--If CI/CD pipeline taking too much time , that is troubleshooting steps and how can we reduce it**

If my CI/CD pipeline is taking too much time, I first identify which stage is consuming the maximum time. I check the Jenkins build history and stage-wise execution time rather than optimizing the entire pipeline blindly.

First, I check the Jenkins agent and its resources, such as CPU, memory, disk I/O, and network connectivity. If the agent is under-resourced, I can move the build to a more suitable agent.

Second, I check the build process and enable caching. For example, I can cache Maven or npm dependencies so that they don't have to be downloaded on every build.

Third, I optimize the Docker build. I use a proper .dockerignore file to avoid sending unnecessary files to the Docker build context, use multi-stage builds to reduce the final image size, and use Docker layer caching where possible.

Finally, I check whether independent pipeline stages can run in parallel, such as unit testing and some security checks. This can reduce the overall pipeline execution time.

After making the changes, I compare the new execution time with the previous build to verify that the optimization actually improved the pipeline.

**2. How do you deploy applications on Amazon EKS? What are the advantages over Amazon ECS?**
Interview Answer

“To deploy an application on EKS, first we containerize the application using Docker and push the image to Amazon ECR.

Then we create Kubernetes manifests such as:

Deployment
Service
ConfigMap
Secret
Ingress

The deployment manifest contains the ECR image and specifies replicas, resources, probes, and other configurations.

For example, after updating the image:

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl rollout status deployment/my-app

EKS then schedules the pods on the worker nodes, and Kubernetes manages scaling, service discovery, health checks, and rolling updates.

EKS advantages over ECS

EKS provides the standard Kubernetes ecosystem and portability.

For example:

Kubernetes-native tooling
Helm
kubectl
Operators
Prometheus/Grafana integration
Kubernetes-based autoscaling
Better portability across cloud environments
Large Kubernetes community and ecosystem

I would choose ECS when the organization wants a simpler AWS-native container orchestration solution with less Kubernetes administration.

I would choose EKS when we need Kubernetes capabilities, ecosystem integrations, or portability.”

**3. What is the difference between Launch Templates and Launch Configurations in Auto Scaling Groups?**
Interview Answer

“Both are used by an Auto Scaling Group to define how EC2 instances should be launched, but Launch Templates are the newer and recommended approach.

Launch Template	Launch Configuration
Recommended by AWS	Legacy approach
Supports versioning	No versioning
Supports newer EC2 features	Limited
Supports multiple instance types	Better support
Supports Spot/On-Demand combinations	Yes
Can be modified/versioned	Immutable
Supports T2/T3 and newer capabilities	Better

For example, in a production environment, I would create a Launch Template containing the AMI, instance type, security groups, IAM instance profile, user data, and storage configuration.

The ASG can then use a specific version of that template.

So, for new implementations I would use Launch Templates rather than Launch Configurations.”

**4. How does an Application Load Balancer differ from a Network Load Balancer? When would you use each?**
Interview Answer

“The main difference is the network layer and type of traffic they handle.

Application Load Balancer

ALB operates at Layer 7, so it understands HTTP/HTTPS traffic.

It supports features such as:

Host-based routing
Path-based routing
HTTP headers
WebSocket
TLS termination
Authentication integrations

For example:

example.com/api → API service
example.com/web → Frontend service

I would use ALB for typical web applications and microservices.

Network Load Balancer

NLB operates at Layer 4 and handles TCP, UDP, and TLS traffic.

It provides:

Very high performance
Low latency
Static IP support
TCP/UDP traffic handling

I would use NLB when I need high-performance Layer-4 load balancing, TCP/UDP traffic, or static IP requirements.

In Kubernetes on EKS, ALB is commonly used with the AWS Load Balancer Controller for HTTP/HTTPS applications, while NLB is useful for Layer-4 services.”

**5. Explain Blue-Green Deployment and Canary Deployment. How have you implemented them?**
Blue-Green Deployment

“In Blue-Green deployment, we maintain two environments.

Blue  → Current production
Green → New version

For example:

ALB
 |
Blue → v1
Green → v2

We deploy the new version to Green and test it.

If everything is healthy, we switch traffic from Blue to Green.

If something goes wrong, we switch traffic back to Blue.

This provides a very quick rollback.”

Canary Deployment

“In Canary deployment, we gradually release the new version to a small percentage of users.

For example:

90% → v1
10% → v2

We monitor metrics such as:

Error rate
Latency
CPU/memory
Application health

If the new version is healthy, we gradually increase traffic:

90/10
→ 70/30
→ 50/50
→ 0/100

For Kubernetes, this can be implemented using ingress/load-balancing capabilities or progressive delivery tools.

Interview closing

“Blue-Green is useful when I need fast switching and rollback, while Canary is useful when I want to reduce the blast radius by gradually exposing users to the new version.”

**6. How do you securely manage application secrets in AWS?**
Interview Answer

“I avoid storing secrets directly in GitHub repositories, Dockerfiles, Jenkinsfiles, or Kubernetes manifests.

For AWS applications, I would use AWS Secrets Manager or AWS Systems Manager Parameter Store.

For example:

Application
    ↓
IAM Role
    ↓
Secrets Manager
    ↓
Database credentials

The application gets permission through an IAM role to retrieve only the required secret.

I also follow least privilege by giving the application access only to the specific secret it requires.

For Kubernetes workloads on EKS, secrets can be integrated using AWS Secrets Manager and appropriate Kubernetes/AWS integration mechanisms.

I also enable encryption using AWS KMS where appropriate and maintain auditing through CloudTrail.”

**7. What are Terraform modules? How do you manage remote state across multiple environments?**
Interview Answer

“Terraform modules are reusable collections of Terraform resources.

Instead of writing the same infrastructure repeatedly, we create reusable modules.

For example:

modules/
 ├── vpc/
 ├── ec2/
 ├── eks/
 └── rds/

Then we can use these modules for different environments.

environments/
 ├── dev/
 ├── staging/
 └── prod/

For remote state, I typically use Amazon S3 as the Terraform state backend and a locking mechanism supported by the current Terraform/AWS backend setup.

The state is stored centrally rather than on an individual engineer's laptop.

For example:

Developer
    ↓
Terraform
    ↓
S3 Remote State
    ↓
AWS Infrastructure

For multiple environments, I keep separate state locations/workspaces or separate backend configurations so that Dev, QA, and Production don't accidentally share the same state.

I also enable versioning and appropriate access controls on the state bucket because Terraform state can contain sensitive infrastructure information.”

**8. How do you troubleshoot a failed Jenkins pipeline or GitHub Actions workflow?**
Interview Answer

“I troubleshoot the pipeline systematically rather than immediately rerunning it.

First, I identify which stage failed.

For example:

Checkout
Build
Test
SonarQube
Docker Build
Trivy
ECR Push
Deployment

Then I check the logs for the exact error.

If checkout fails

I check:

Repository URL
Credentials
Branch
GitHub connectivity
Webhook
If build fails

I check:

Maven/npm errors
Dependencies
Java/Node version
Build configuration
If Docker build fails

I check:

Dockerfile
Build context
Dependencies
Docker daemon
If ECR push fails

I check:

AWS credentials/role
IAM permissions
ECR repository
AWS region
Docker authentication
If EKS deployment fails

I check:

kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events
kubectl rollout status deployment/<deployment>

I fix the root cause and then rerun only when appropriate.

“My approach is to identify the failed stage, analyze the logs, verify the underlying dependency, fix the root cause, and validate the pipeline again.”

**9. Explain IAM Roles, Policies, and Cross-Account Role Assumption with a real-world example.**
Interview Answer

“An IAM policy defines what actions are allowed or denied.

An IAM role is an identity that can be assumed by AWS services, users, or identities from another account.

For example, suppose we have:

Account A → Jenkins
Account B → Production AWS

Jenkins runs in Account A, but it needs to deploy an application to EKS in Account B.

Instead of creating permanent access keys for the production account, we create a role in Account B.

Jenkins
  ↓
STS AssumeRole
  ↓
Production Deployment Role
  ↓
EKS / ECR / other required resources

The trust policy on the production role allows the trusted Jenkins identity to assume the role.

The permissions policy attached to that role grants only the required permissions.

This provides least privilege and temporary credentials instead of storing permanent production AWS keys.”

**10. How do you monitor AWS infrastructure using CloudWatch? What metrics and alarms do you configure?**
Interview Answer

“I use Amazon CloudWatch to monitor AWS infrastructure and configure alarms for important resource and application metrics.

For EC2, I monitor:

CPUUtilization
Network traffic
Disk-related metrics where available/configured
StatusCheckFailed
Memory utilization using the CloudWatch Agent

For RDS:

CPUUtilization
DatabaseConnections
FreeStorageSpace
FreeableMemory
Read/Write latency
IOPS

For ALB:

RequestCount
TargetResponseTime
HTTP 4xx
HTTP 5xx
UnhealthyHostCount

For EKS, I monitor cluster/node/pod health using appropriate CloudWatch integrations and also use Prometheus/Grafana when detailed Kubernetes/application metrics are required.

For example, if EC2 CPU remains above a threshold for a defined period, CloudWatch can trigger an alarm and send notifications through SNS or trigger an automated action where appropriate.”

**11. How do you optimize AWS costs for EC2, EKS, S3, and RDS?**
Interview Answer

“I use a combination of right-sizing, utilization analysis, automation, and appropriate pricing models.

EC2
Right-size instances
Remove unused instances
Use Auto Scaling
Use Reserved Instances/Savings Plans where workloads are predictable
Use Spot Instances for suitable workloads
Schedule non-production environments
EKS
Right-size worker nodes
Use Cluster Autoscaler/Karpenter where appropriate
Use Spot capacity for fault-tolerant workloads
Configure appropriate pod resource requests and limits
Remove unused workloads and nodes
S3
Enable lifecycle policies
Move infrequently accessed data to suitable storage classes
Delete unnecessary objects
Use Intelligent-Tiering where appropriate
Review incomplete multipart uploads
RDS
Right-size DB instances
Use Reserved Instances for predictable workloads
Delete unused databases/snapshots where appropriate
Configure storage appropriately
Use read replicas only when justified

I also use AWS Cost Explorer and AWS Budgets to continuously monitor spending.”

**12. What happens when an EC2 instance in an Auto Scaling Group becomes unhealthy?**
Interview Answer

“When an EC2 instance in an Auto Scaling Group becomes unhealthy, the ASG detects the unhealthy state through EC2 or configured load balancer health checks.

The ASG marks the instance as unhealthy and terminates it.

Then the Auto Scaling Group launches a replacement instance using the configured Launch Template.

The new instance goes through the configured bootstrap/user-data process and is registered with the load balancer once it passes the required health checks.

So the flow is:

Unhealthy Instance
       ↓
ASG detects failure
       ↓
Terminate instance
       ↓
Launch replacement
       ↓
Health checks
       ↓
Register with Target Group
       ↓
Receive traffic

This provides self-healing capability for the application infrastructure.”

**13. How do you roll back a failed Kubernetes deployment?**
Interview Answer

“If a new Kubernetes deployment is unhealthy, first I check the rollout status and pod logs.

For example:

kubectl rollout status deployment/my-app
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>

If the previous version is known to be stable, I can rollback using:

kubectl rollout undo deployment/my-app

Then I verify:

kubectl rollout status deployment/my-app
kubectl get pods

I also check application health and monitoring dashboards.

Kubernetes maintains ReplicaSets for previous revisions, which makes rollback possible.

In a production incident, I prioritize restoring service first, then investigate the root cause and implement a permanent fix.”

**14. How do you scan Docker images for vulnerabilities before deployment?**
Interview Answer

“I use Trivy as part of the CI/CD pipeline.

After building the Docker image, Jenkins runs a Trivy scan.

For example:

trivy image myapp:${BUILD_NUMBER}

The scan checks for vulnerabilities in:

OS packages
Application dependencies
Libraries
Known CVEs

We configure security gates based on severity.

For example:

Docker Build
     ↓
Trivy Scan
     ↓
CRITICAL/HIGH vulnerabilities
     ↓
Pipeline fails

If a critical vulnerability is detected, I don't simply ignore it. I first determine whether it is exploitable and whether a patched base image or dependency is available.

After remediation, I rebuild and rescan the image.

This prevents vulnerable images from reaching ECR/EKS.”

**15. Describe a major production incident you handled. What was the root cause, and how did you resolve it?**
Interview Answer

“One production incident I handled involved a Kubernetes deployment where newly deployed pods started going into CrashLoopBackOff, which affected application availability.

We first received an alert from our monitoring system indicating that the application pods were repeatedly failing.

I started troubleshooting by checking:

kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events

The logs showed that the application was failing during startup because of an incorrect configuration/environment variable introduced with the new deployment.

As an immediate mitigation, we rolled back the Kubernetes deployment to the previous stable version:

kubectl rollout undo deployment/<deployment-name>

We then verified the rollout and application health. Once the service was stable, we investigated the deployment configuration and identified the incorrect environment variable as the root cause.

Root Cause

The root cause was an incorrect application configuration introduced during deployment.

Resolution

We:

Rolled back to the stable version.
Corrected the configuration.
Tested the fix in the lower environment.
Redeployed the corrected version.
Monitored the application after deployment.
Preventive Actions

We added additional configuration validation and deployment checks to the CI/CD process and improved monitoring around pod failures.

I also documented the incident in an RCA/post-incident report, including the timeline, impact, root cause, resolution, and corrective/preventive actions.

The main lesson was that fast rollback restores service, but the RCA and preventive action are necessary to ensure the same issue doesn't happen again.”

**⭐ One-line architecture to remember for interviews**

Memorize this flow:

Developer → GitHub → PR/Review → Webhook → Jenkins → Build → Unit Test → SonarQube → OWASP → Docker Build → Trivy → ECR → EKS → ALB → Prometheus/Grafana + CloudWatch

And for AWS infrastructure:

Terraform → VPC → ALB → EKS/EC2 → ECR → RDS/S3 → CloudWatch

This gives you a strong end-to-end story instead of answering each technology in isolation.

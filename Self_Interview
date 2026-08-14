# DevOps Interview Questions & Answers

**Good afternoon, and thank you for giving me this opportunity.**

My name is Vikash Kumar. I have around 12 years of total IT experience, including over 6 years of hands-on experience as a DevOps Engineer. Currently, I am working as a DevOps Engineer at Flexis IT Pvt. Ltd. in Delhi.

I began my career in Desktop Support and System Administration -to supporting Linux and Windows both environments, where I built a strong foundation in IT operations, Linux administration and infrastructure management.

From there, I moved into DevOps, where I've worked with designing CI/CD pipelines with Jenkins and GitHub Action, containerizing and orchestrating applications with Docker and Kubernetes, automating infrastructure with Terraform and Ansible and monitoring systems with Prometheus and Grafana.

On the cloud side, I primarily work with AWS, including EC2, Lambda, EKS, IAM, VPC, Route 53, Auto Scaling, ELB, CloudWatch, and S3, helping teams build scalable, secure, and highly available environments.

One project at my current organization, I'm proudy say, where I had the opportunity to redesign the entire release process end-to-end. I implemented a multi-stage Dockerfile to optimize image size and build efficiency, and I integrated SonarQube, OWASP Dependency-Check, and Trivy into the CI/CD pipeline to detect code quality issues and security vulnerabilities early in the development lifecycle, preventing them from reaching production.

I've also built full observability into our production systems using Prometheus and Grafana, which shifted our team from reacting to outages to catching problems before users ever noticed.

Throughout my DevOps journey, I've worked closely with Development, QA, and Operations teams that enables faster and smoother software delivery.And I'm someone who believes learning never really ends in this field — there's always a new tool or a better way to do something.

With my strong technical background and hands-on experience, I'm confident I can add real value to your DevOps team."

Thank you.

--------------------------------------------------------------------------------------------------------------------------------------------------

**Your Roles and responsibilities.**

As a DevOps Engineer, my primary responsibility is to automate the complete software delivery process and ensure the application is reliable, secure, and highly available.

My Roles and responsibilities include:

Managing source code using Git and GitHub.
Building and maintaining CI/CD pipelines using Jenkins and GitHub Actions for build, test, security scanning, and deployment.
Creating and maintaining Docker images and deploying applications to Kubernetes.
Provisioning and managing AWS infrastructure using Terraform, including EC2, EKS, VPC, IAM, Route 53, and Auto Scaling.
Integrating SonarQube, OWASP Dependency Check, and Trivy into the CI/CD pipeline to improve code quality and security.
Monitoring applications and infrastructure using Prometheus and Grafana, and configuring alerts for critical services.
Troubleshooting build failures, deployment issues, and production incidents, and performing Root Cause Analysis (RCA) to prevent similar issues in the future.
Working closely with Development, QA, and Operations teams to ensure smooth and reliable releases.
Participating in production deployments, change requests, and on-call support whenever required.

One of my major contributions was redesigning our CI/CD pipeline, which reduced deployment time by around 40% and reduced deployment failures by 25%, making the release process faster and more reliable.

Overall, my focus is on automation, infrastructure reliability, application availability, and continuous improvement.
-----------------------------------------------------------------------------------------------------------------------------------------------------
**Can you explain your CI/CD pipeline step by step?**

In my current project, we use GitHub, Jenkins, Docker, Kubernetes, SonarQube, Nexus, Trivy, Prometheus, Grafana, and AWS to automate the complete software delivery process.

The pipeline works as follows:

Step 1: Developer Commits Code

Developers push their code to the GitHub repository using feature branches. After creating PR and the code review, it is merged into the main branch.

Step 2: Jenkins Trigger

Once the code is merged, GitHub Webhook automatically triggers the Jenkins pipeline.

Step 3: Checkout Source Code

Jenkins pulls the latest source code from the GitHub repository.

git checkout main
Step 4: Build the Application

Depending on the application, Jenkins builds it.

For Java applications:

mvn clean package

This creates the application artifact (JAR/WAR).

Step 5: Code Quality Analysis

Next, Jenkins runs SonarQube Scan.

This checks

Code Quality
Bugs
Code Smells
Vulnerabilities
Security Hotspots

If the Quality Gate fails, the pipeline stops automatically.

Step 6: Dependency Scan

After SonarQube, we run OWASP Dependency Check.

It scans third-party libraries for known CVEs.

If critical vulnerabilities are found, deployment is stopped.

Step 7: Build Docker Image

If everything passes,

Jenkins builds a Docker image.

Example:

docker build -t ecommerce:v1 .
Step 8: Scan Docker Image

Next, we scan the Docker image using Trivy.

It checks

OS vulnerabilities
Package vulnerabilities
Secrets
Misconfigurations

Only secure images are allowed for deployment.

Step 9: Push Image to Repository

The Docker image is pushed to the image repository.

For example:

docker push repo/ecommerce:v1

We also upload build artifacts to Nexus Repository for version management.

Step 10: Deploy to Kubernetes

Jenkins deploys the application to the Kubernetes cluster.

Example:

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

For production, we usually perform a Rolling Update so there is no downtime.

Step 11: Verification

After deployment,

we verify

Pods are running
Services are accessible
Health checks are successful
Application is reachable

Commands:

kubectl get pods
kubectl get svc
kubectl describe pod

Step 12: Monitoring

Finally,

Prometheus collects metrics,

Grafana displays dashboards,

and Alertmanager sends alerts if CPU, Memory, Disk, or Application Health crosses predefined thresholds.



Pipeline Flow
Developer
      ↓
GitHub
      ↓
Webhook
      ↓
Jenkins
      ↓
Checkout Code
      ↓
Build (Maven)
      ↓
SonarQube
      ↓
OWASP Dependency Check
      ↓
Docker Build
      ↓
Trivy Scan
      ↓
Push Image (Docker Registry)
      ↓
Store Artifact (Nexus)
      ↓
Deploy to Kubernetes
      ↓
Health Check
      ↓
Prometheus + Grafana Monitoring
---------------------------------------------------------------------------------------------------------------------------------------
**How do you handle production incidents?**

Whenever a production incident occurs, my first priority is to restore the service as quickly as possible while minimizing business impact.

First, I check alerts from Prometheus and Grafana to understand what triggered the issue.

Then I verify:

Application logs
Kubernetes pod status
Jenkins deployment history
AWS resources such as EC2, EKS, and Load Balancer
CPU, Memory, and Disk utilization

If the issue is related to a recent deployment, I immediately roll back to the previous stable version.

If it's an infrastructure issue, I identify the root cause and fix it accordingly.

Throughout the incident, I keep all stakeholders informed and work closely with the Development, QA, and Operations teams.

Once the issue is resolved, I perform a Root Cause Analysis (RCA), document the findings, and implement preventive measures so the same issue doesn't happen again.
------------------------------------------------------------------------------------------------------------------------------------------------------
**How do you implement monitoring and alerting?**

In my current project, we use Prometheus and Grafana for monitoring and alerting.

Prometheus collects metrics from applications, Kubernetes clusters, and servers by scraping configured targets at regular intervals.

We monitor important metrics such as:

CPU usage
Memory utilization
Disk usage
Pod health
Node health
Application response time
HTTP error rates
Network traffic

Grafana is connected to Prometheus and provides dashboards to visualize these metrics in real time.

We configure alert rules in Prometheus and use Alertmanager to send notifications through Email, Slack, or Microsoft Teams whenever predefined thresholds are exceeded.

-----------------------------------------------------------------------------------------------------------------------------------------------
**Could you please tell me the Git branching strategy used in your company?**

Yes. In my current project, we follow the Git Feature Branch Workflow with separate branches for development, testing, and production.

Our branching strategy is as follows:

main (or master): This is our production branch. Only stable and tested code is merged into this branch, and every production release is tagged with a version number.
develop: This is the integration branch where all completed features are merged. It always contains the latest development code and is used for deployment to the development environment.

feature branches: Whenever a developer starts working on a new feature or user story, they create a feature branch from the develop branch, for example:

feature/login
feature/payment

After completing the work, they raise a Pull Request (PR). The code is reviewed, and after approval, it is merged back into the develop branch.

release branch: When we are ready for a release, we create a release branch from develop, such as:

release/1.2.0

Only bug fixes and release-related changes are allowed in this branch. After successful QA testing, it is merged into both main and develop.

hotfix branch: If a critical issue is found in production, we create a hotfix branch directly from main, for example:

hotfix/login-fix

After fixing and testing the issue, it is merged back into both main and develop so that all branches remain synchronized.

Before merging any branch, we follow a mandatory Pull Request (PR) process. At least one team member reviews the code, and the CI pipeline runs automatically to perform:

Build validation
Unit tests
SonarQube code quality checks
Security scans

Only after all checks pass is the code merged.

This branching strategy helps us maintain code quality, avoid conflicts, and support parallel development by multiple developers.

Branch Flow
main (Production)
      ▲
      │
 release
      ▲
      │
develop
  ▲   ▲   ▲
  │   │   │
feature feature feature

-----------------------------------------------------------------------------------------------------------------------------------------------

**Tell me about your daily routine.**

In my current role, my day usually starts by checking Microsoft Teams and my emails to see if there are any important updates, production issues, maintenance activities, or priority tasks assigned to me.

After that, I check the health of our production environment. I review Prometheus and Grafana dashboards to make sure all applications and infrastructure are running smoothly. I also check for any critical alerts or overnight production issues that need immediate attention.

Next, I attend the daily stand-up meeting with the Development, QA, and Product teams, where we discuss the previous day's work, today's tasks, and any blockers.
After that, I monitor Jenkins and GitHub Actions pipelines, troubleshoot build or deployment issues, manage Kubernetes deployments, and provision infrastructure using Terraform on AWS. I also support production deployments, resolve incidents, collaborate with Development and QA teams, and work on automation and continuous improvements. Before the end of the day, I update my work in JIRA and document any important changes.

------------------------------------------------------------------------------------------------------------------------------------------------

**🚀 Deploy Java Application on AWS — 3-Tier Architecture******

<p align="center">
  <img src="architecture-diagram.png" alt="AWS 3-Tier Architecture" width="100%">
</p>

<p align="center">
  <b>A hands-on AWS Cloud project demonstrating the deployment of a Java web application using a secure 3-tier architecture.</b>
</p>

<p align="center">
  AWS • EC2 • VPC • ALB • Auto Scaling • RDS • Nginx • Tomcat • Java • MySQL • Linux
</p>

---

## 📌 Project Overview

This project demonstrates the deployment of a Java web application on AWS using a **3-tier architecture**.

The architecture separates the environment into:

- **Presentation / Load Balancing Layer** — Application Load Balancer
- **Application Layer** — EC2 instances running Nginx, Tomcat and the Java application
- **Database Layer** — Amazon RDS for MySQL

The application servers are placed in **private application subnets**, while the Application Load Balancer is deployed in **public subnets**.

The database is placed in private database subnets and is accessible only from the application layer through controlled Security Group rules.

The project also includes **Auto Scaling**, health checks, failure testing, private networking and AWS Systems Manager-based instance administration.

---

# 🏗️ Architecture

## AWS Architecture

The project uses the following VPC design:

```text
VPC
10.0.0.0/16

├── Availability Zone A
│   │
│   ├── Public Subnet A
│   │   └── 10.0.0.0/24
│   │
│   ├── App Subnet A
│   │   └── 10.0.1.0/24
│   │
│   └── DB Subnet A
│       └── 10.0.2.0/24
│
└── Availability Zone B
    │
    ├── Public Subnet B
    │   └── 10.0.3.0/24
    │
    ├── App Subnet B
    │   └── 10.0.4.0/24
    │
    └── DB Subnet B
        └── 10.0.5.0/24

Main traffic path
Internet
    │
    ▼
Internet Gateway
    │
    ▼
Application Load Balancer
    │
    ▼
Private EC2 Application Instances
    │
    ├── Nginx :80
    │
    └── Tomcat :8080
            │
            ▼
      Java Application
            │
            ▼
     Amazon RDS MySQL
          :3306
Architecture Diagram

The complete architecture is shown below:

<p align="center"> <img src="architecture-diagram.png" alt="AWS 3-Tier Architecture Diagram" width="100%"> </p>

🔄 Application Request Flow

The application request follows this path:
1. User
     │
     ▼
2. Application Load Balancer
     │
     ▼
3. EC2 Application Instance
     │
     ▼
4. Nginx :80
     │
     ▼
5. Tomcat :8080
     │
     ▼
6. Java Application
     │
     ▼
7. Amazon RDS MySQL :3306
     │
     ▼
8. Java Application
     │
     ▼
9. Response to User
Step-by-step

1. User

The user accesses the application through the Application Load Balancer.

2. Application Load Balancer

The ALB receives the HTTP request and forwards it to a healthy application target.

3. EC2 Application Instance

The request reaches an EC2 instance located in a private application subnet.

4. Nginx

Nginx listens on port 80 and acts as a reverse proxy.

5. Tomcat

Nginx forwards the request to Tomcat on port 8080.

6. Java Application

Tomcat serves the Java web application.

7. Amazon RDS

When database access is required, the Java application connects to Amazon RDS MySQL over port 3306.

8. Application Response

The database result is returned to the Java application.

9. User Response

The response travels back through the application server and ALB to the user.

☁️ AWS Services Used
| AWS Service               | Purpose                                                 |
| ------------------------- | ------------------------------------------------------- |
| Amazon VPC                | Provides the isolated network environment               |
| Internet Gateway          | Provides internet connectivity for public subnets       |
| Amazon EC2                | Hosts the application servers                           |
| Application Load Balancer | Distributes HTTP traffic to healthy application targets |
| Auto Scaling Group        | Maintains application instance capacity                 |
| Launch Template           | Defines the EC2 instance configuration                  |
| Amazon RDS                | Provides managed MySQL database                         |
| Security Groups           | Controls inbound and outbound network access            |
| Route Tables              | Controls subnet traffic routing                         |
| IAM                       | Provides AWS permissions for resources                  |
| AWS Systems Manager       | Used for instance administration                        |

🌐 VPC & Networking
VPC
VPC CIDR: 10.0.0.0/16
Region: ap-south-1
Availability Zone A
The VPC was divided into six subnets across two Availability Zones.
| Subnet          | CIDR          | Purpose                     |
| --------------- | ------------- | --------------------------- |
| Public Subnet A | `10.0.0.0/24` | Public-facing AWS resources |
| App Subnet A    | `10.0.1.0/24` | Application EC2 instances   |
| DB Subnet A     | `10.0.2.0/24` | Database subnet             |

Availability Zone B
| Subnet          | CIDR          | Purpose                     |
| --------------- | ------------- | --------------------------- |
| Public Subnet B | `10.0.3.0/24` | Public-facing AWS resources |
| App Subnet B    | `10.0.4.0/24` | Application EC2 instances   |
| DB Subnet B     | `10.0.5.0/24` | Database subnet             |

🛣️ Route Tables
Public Route Table

The public route table contains:

Destination       Target

10.0.0.0/16       local
0.0.0.0/0         Internet Gateway

This allows resources in the associated public subnets to communicate with the internet through the Internet Gateway.

Private Route Table

The private route table contains:

Destination       Target

10.0.0.0/16       local

The application and database subnets do not have a direct route to the internet.

🔐 Security Groups

Security Groups were used to control communication between the different layers.

1. ALB Security Group
Inbound:

HTTP 80 → 0.0.0.0/0

The Application Load Balancer accepts HTTP traffic from the internet.

2. Application Security Group
Inbound:

HTTP 80 → ALB Security Group
SSH 22   → Administrator IP (/32)

Port 8080 is not exposed directly to the internet.

Tomcat traffic is reached through Nginx.

3. Database Security Group
Inbound:

MySQL 3306 → Application Security Group

The database does not accept direct public internet traffic.

Security Flow
Internet
    │
    │ HTTP :80
    ▼
ALB Security Group
    │
    │ HTTP :80
    ▼
Application Security Group
    │
    │ MySQL :3306
    ▼
Database Security Group

This creates a controlled communication path between the application layers.

⚖️ Application Load Balancer

An Application Load Balancer was deployed across the two public subnets.

                    Internet
                       │
                       ▼
             ┌───────────────────┐
             │ Application Load  │
             │ Balancer          │
             └─────────┬─────────┘
                       │
                Target Group
                  ┌────┴────┐
                  ▼         ▼
                EC2-A     EC2-B
ALB Configuration
Scheme: Internet-facing
IP type: IPv4
Listener: HTTP :80
Target type: Instance
Target port: 80

The ALB forwards requests to the application target group.

❤️ Target Group & Health Check

The application target group was configured to check the application through:

Protocol: HTTP
Port: 80
Health Check Path:

/dptweb-1.0/login

The ALB sends traffic only to targets that pass the configured health check.

🔄 Auto Scaling Group

The application layer was managed using an Auto Scaling Group.

During implementation
Minimum: 1
Desired: 2
Maximum: 2

The Auto Scaling Group used a Launch Template to define the EC2 configuration.

The application instances were distributed across the two application subnets.

Failure Testing

An application EC2 instance managed by the Auto Scaling Group was intentionally terminated.

The expected behavior was:

EC2 instance terminated
        │
        ▼
ASG detects reduced capacity
        │
        ▼
Replacement EC2 launched
        │
        ▼
Application starts
        │
        ▼
Target passes health check
        │
        ▼
ALB sends traffic to healthy target

This test helped verify the relationship between:

Auto Scaling Group
        +
Launch Template
        +
Target Group
        +
Application Load Balancer
Current Cost-Control State

After completing the project, the Auto Scaling Group was scaled down to control AWS costs:

Minimum: 0
Desired: 0
Maximum: 2

Application instances are therefore not continuously running.

🖥️ Application Server Stack

Each application server uses:

Amazon Linux 2023
        │
        ▼
      Nginx
      :80
        │
        ▼
     Tomcat
     :8080
        │
        ▼
 Java Application
Nginx

Nginx is used as a reverse proxy.

It receives traffic on port 80 and forwards application requests to Tomcat.

Tomcat

Apache Tomcat runs the Java web application on port 8080.

Java Application

The Java application is deployed on Tomcat and communicates with Amazon RDS MySQL when database access is required.

🗄️ Amazon RDS MySQL

Amazon RDS for MySQL was used as the database layer.

Application-to-database communication:

EC2 Application Server
        │
        │ TCP :3306
        ▼
Amazon RDS MySQL

The RDS instance is deployed in a private database subnet.

The database Security Group allows MySQL traffic only from the application Security Group.

RDS Configuration Note

The DB subnet group spans the two database subnets:

DB Subnet A
10.0.2.0/24

DB Subnet B
10.0.5.0/24

However, the actual RDS database instance used for this hands-on implementation was deployed as a single-AZ instance.

🔑 Private EC2 Administration

The application servers were placed in private subnets.

For administration, AWS Systems Manager was configured using the:

AmazonSSMManagedInstanceCore

IAM policy through the EC2 instance role.

During troubleshooting, controlled SSH access and local port forwarding were also used for administration.

The private application instances were not intended to be directly exposed to the public internet.

🧪 Failure & Troubleshooting

This project involved hands-on troubleshooting rather than only following a deployment guide.

Some of the issues investigated during implementation included:

Java and Maven version mismatch
Missing MySQL JDBC driver
Tomcat dependency conflict
Tomcat service configuration
Tomcat service availability after reboot
EC2 SSH connectivity
Security Group configuration
AWS Systems Manager configuration
RDS connectivity
RDS temporary instance capacity issue
Application Load Balancer target health
Auto Scaling instance replacement
Private subnet administration
Application-to-RDS connectivity
🔧 Example Troubleshooting Approach

When an ALB target becomes unhealthy, the troubleshooting path is:

ALB Target Unhealthy
        │
        ▼
Check Target Health Reason
        │
        ▼
Check Nginx
        │
        ▼
Check Tomcat
        │
        ▼
Check Java Application
        │
        ▼
Check Application Logs
        │
        ▼
Check RDS Connectivity
        │
        ▼
Verify Application Health Endpoint

This layered troubleshooting approach helps identify whether the problem is related to:

Networking
Security Groups
Nginx
Tomcat
Java application
Database
Load Balancer health checks
🛠️ Technologies Used
Cloud
AWS
Amazon VPC
Amazon EC2
Application Load Balancer
Auto Scaling Group
Launch Template
Amazon RDS
IAM
AWS Systems Manager
Application
Java
Apache Tomcat
Nginx
MySQL
Operating System
Amazon Linux 2023
Networking
VPC
CIDR
Public Subnets
Private Subnets
Route Tables
Internet Gateway
Security Groups
TCP/IP
📚 Key Learning Outcomes

Through this project, I gained hands-on experience with:

Designing a 3-tier AWS architecture
Creating a VPC using CIDR 10.0.0.0/16
Creating public and private subnets
Understanding subnet CIDR allocation
Configuring route tables
Working with Internet Gateway
Designing Security Group rules
Deploying EC2 application servers
Installing and configuring Nginx
Configuring Tomcat
Deploying a Java application
Configuring Application Load Balancer
Creating Target Groups
Configuring health checks
Creating Launch Templates
Configuring Auto Scaling Groups
Testing EC2 failure and replacement
Connecting EC2 to RDS MySQL
Working with private application and database subnets
Using AWS Systems Manager
Troubleshooting AWS networking and application issues
Monitoring AWS costs
💰 Cost Management

This project was created as a hands-on learning and portfolio project.

After completing the implementation, resources were intentionally scaled down or stopped to control ongoing AWS costs.

Current cost-control actions include:

Auto Scaling Group desired capacity set to 0
Application EC2 instances terminated through the Auto Scaling Group
RDS instance stopped
Project VPC and supporting architecture retained for future demonstrations
AWS Billing and Cost Explorer used to monitor project spending

⚠️ AWS resources can still incur charges even when application instances are stopped. Always check AWS Billing and Cost Explorer before leaving resources running.

📊 Project Status
Status: Completed
Type: Hands-on AWS Cloud Project
Environment: AWS
Region: ap-south-1

The architecture was successfully implemented and tested.

Failure scenarios and troubleshooting exercises were performed during the implementation.

The project is currently kept in a cost-controlled idle state for future demonstrations and interview discussions.

🎯 What This Project Demonstrates

This project demonstrates practical understanding of:

AWS Networking
      +
Linux Administration
      +
EC2
      +
Load Balancing
      +
Auto Scaling
      +
Nginx
      +
Tomcat
      +
Java Application
      +
RDS MySQL
      +
Security Groups
      +
Troubleshooting

The main focus was not simply deploying an application, but understanding how the different infrastructure and application layers communicate with each other.

👨‍💻 Author
Mohamed Sarbudeen

Cloud & DevOps Engineer

Focused on:

AWS
Linux
Kubernetes
Docker
Terraform
CI/CD
Monitoring
Cloud Operations

⭐ If you find this project useful, feel free to explore the repository.

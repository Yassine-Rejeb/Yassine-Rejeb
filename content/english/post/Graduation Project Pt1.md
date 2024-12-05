+++
author = "M0D4S"
title = "Chaos Engineering Exploration: Developping a resilient web application for file and secret sharing."
date = "2024-04-28"
description = "Introducing the project and its phases and discussing the first phase"
tags = [
    "ChaosEngineering", 
    "SoftwareReliability", 
    "WebDevelopment", 
    "InternshipProject", 
    "CloudComputing", 
    "FileSharingApp", 
    "SecretManagement", 
    "DjangoFramework", 
    "VueJS", 
    "AgileDevelopment", 
    "ScrumMethodology", 
    "SoftwareArchitecture", 
    "DevOps", 
    "FullStackDevelopment", 
    "SoftwareEngineering", 
    "SystemResilience"
]
+++
## Project Introduction & Development Phase

### Introduction
Chaos Engineering is rapidly becoming a cornerstone in enhancing software reliability. In this blog post, I’ll introduce the **engineering deg graduation internship project** I worked on, which revolved around developing a **file and secret sharing web application** for the purpose of exloring Chaos Engineering principles to improve system resilience. The journey included **web development**, **containerization**, **Deployment on K8S with monitoring (Test/Dev cluster & AKS)** and finally experimenting with **Chaos Enginering**.  
The project is published on my [GitHub](https://github.com/Yassine-Rejeb/fssp).

---

### Why This Project?
Modern software systems are increasingly more distributed and more cloud based and with that comes inherent complexity (This was researched as early as 1986 by Frederick P. Brook Jr. in his research paper **No silver bullet - essence and accidents of software engineering**). 
Traditional methods like **manual testing** or **reactive troubleshooting** often fall short in identifying mishaps of software engineering. Chaos engineering emerged to solve that by relying on chaos theory. 
This project aimed to: 
- Build a **resilient web application** for file and secret sharing.
- Demonstrate the effectiveness of **Chaos Engineering** in identifying and addressing hidden system weaknesses.

---

### Project Scope and Objectives
The project was structured to achieve the following objectives:
1. **Develop a secure web application** with robust user management, file, and secret sharing features.
2. **Containerize the application** to ensure portability and consistency.
3. Deploy the application on **Azure Kubernetes Service (AKS)** for production readiness with the use of Key Vault and monitoring services.
4. Implement **Chaos Engineering experiments** to test and enhance the application's resilience.

---

### Project Planning

#### Global Use Case Diagram
As a starting point, we need to have an idea about the use cases of our application, so here is the general use case diagram:
![General Use Case Diagram of the Web App](/images/portfolio/fssp/global_uc.png)

**1. Actors:** 
* **Guest:** A user who is not logged into the system.
Primary interaction: They can create an account.
* **Registered User:** A user who has created an account in the system and has
access to various features.
* **Primary interactions:** Login, Manage files, manage secrets, manage their
account, and log out.
* **Azure Key Vault (Cloud):** A cloud service that the system interacts with to
create and manage cryptographic key pairs for the sake of encrypting secrets.
No direct interaction with human actors.

**2. Use Cases:**
* **Create Account (Guest):** The guest can create a new account to become a
registered user. This process includes the creation of a key pair via the Azure
Key Vault, which is necessary for secure file and secret management.
* **Verify Account (Guest):** The system includes a step for account verification.
This use case is triggered as part of the account creation process.
* **Login (Registered User):** A registered user logs into the system. This use
case is crucial as it allows access to the system’s core functionality like file and
secret management.
* **Logout (Registered User):** After interacting with the system, the registered
user can log out to terminate the session.
* **Manage Files (Registered User):** Once logged in, the user can perform
file-related actions (e.g., uploading, downloading, or sharing files).
* **Manage Secrets (Registered User):** Similarly, the user can manage secrets
(e.g., store, retrieve, or update sensitive information).
* **Manage Account (Registered User):** Users can modify their account set-
tings or update personal information after logging in.


#### Project Management Approach
The whole project was carried out in **agile sprints** using the Scrum methodology. Below is an overview of the key releases/sprints:

| **Release**   | **Sprint**    | **Details**                                                                                                            |
|---------------|---------------|------------------------------------------------------------------------------------------------------------------------|
| **Release 1** | **Sprint 1**  | Development of the User Management feature.                                                                            |
|               | **Sprint 2**  | Development of the Secret Management feature.                                                                          |
|               | **Sprint 3**  | Containerization of the application so far.                                                                            |
| **Release 2** | **Sprint 1**  | Setup of a test Kubernetes cluster and deployment of containerized services.                                           |
|               | **Sprint 2**  | Development of the File Management feature, requiring a running Kubernetes cluster. Feature containerized and deployed on the test cluster. |
|               | **Sprint 3**  | Setup of monitoring tools (Prometheus, Grafana) using Istio for service mesh capabilities.                             |
| **Release 3** | **Sprint 1**  | Setup of Azure Kubernetes Service (AKS) and managed Istio. Managed Prometheus and Grafana were integrated for monitoring. Azure Key Vault was implemented and deployment of all created manifests in AKS. |
|               | **Sprint 2**  | Setup of a chaos engineering environment to test the resilience of the system, with experiments conducted to identify and address system weaknesses. |


### What’s Next?
With the development phase complete, the next step involved **containerizing the application** and deploying it to a **test Kubernetes cluster**. Stay tuned for the next post, where I’ll discuss this phase in detail!

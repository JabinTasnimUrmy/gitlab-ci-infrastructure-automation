# 🚀 Automated CI/CD Pipeline & Infrastructure as Code

![GitLab](https://img.shields.io/badge/gitlab-%23181717.svg?style=for-the-badge&logo=gitlab&logoColor=white)
![Ansible](https://img.shields.io/badge/ansible-%23EE0000.svg?style=for-the-badge&logo=ansible&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Vagrant](https://img.shields.io/badge/vagrant-%231563FF.svg?style=for-the-badge&logo=vagrant&logoColor=white)
![Java](https://img.shields.io/badge/java-%23ED8B00.svg?style=for-the-badge&logo=openjdk&logoColor=white)

## 📖 Project Overview

This project demonstrates the implementation of a **complete DevOps lifecycle**, from provisioning a self-hosted integration server to deploying a Java application via a CI/CD pipeline.

Instead of using pre-configured cloud services, I built the infrastructure from scratch to understand the underlying mechanics of **Infrastructure as Code (IaC)** and **Containerization**.

**Key Achievements:**
*   **Provisioning:** Automated the setup of a Ubuntu VM using **Vagrant** and **VirtualBox**.
*   **Configuration Management:** Used **Ansible** playbooks to install GitLab, Docker, and configure system dependencies without manual intervention.
*   **CI/CD:** Designed a 4-stage GitLab CI pipeline to build, test, run, and archive a Java Maven application.

---

## 🛠️ Tech Stack & Tools

*   **Infrastructure:** Oracle VirtualBox, Vagrant
*   **Configuration Management:** Ansible (Playbooks, Roles, Handlers)
*   **CI/CD Platform:** GitLab CE (Self-Hosted)
*   **Runners:** GitLab Runner (utilizing Docker executor)
*   **Application:** Java, Maven (Hello World application)
*   **Containerization:** Docker

---

## ⚙️ Architecture & Implementation

### 1. Infrastructure as Code (Vagrant + Ansible)
The environment is provisioned automatically using `vagrant up`. This triggers an Ansible playbook (`playbook.yml`) which performs the following tasks:
*   **Dependency Management:** Installs `curl`, `openssl`, `tzdata`, and `ca-certificates`.
*   **GitLab Installation:** Downloads and installs the GitLab Enterprise Edition script and package.
*   **Docker Configuration:** Installs Docker and adds the `vagrant` user to the docker group for permission management.
*   **Automated Configuration:** Modifies `/etc/gitlab/gitlab.rb` to set the `external_url` and custom ports.

### 2. The CI/CD Pipeline (`.gitlab-ci.yml`)
I created a multi-stage pipeline that triggers automatically on every code push.

| Stage | Description |
| :--- | :--- |
| **Build** | Uses the `maven:latest` Docker image to compile the Java source code (`mvn compile`). |
| **Test** | Runs unit tests using the Maven Surefire plugin. Ensures code quality before deployment. |
| **Run** | Executes the packaged `.jar` file within the pipeline to verify runtime functionality. |
| **Deploy** | **(Crucial Step)** Extracts the built `.jar` file and stores it as a permanent **GitLab Artifact**, allowing users to download the final binary. |

---

## 🧠 Challenges & Solutions

During this project, I encountered real-world engineering challenges. Here is how I solved them:

### 🐛 Challenge 1: Network & IP Conflicts
**The Issue:** The Vagrant VM was assigning a dynamic IP (`192.168.56.9`) that did not match the static IP I initially configured in the GitLab settings (`192.168.33.9`), making the UI inaccessible.
**The Solution:** 
1. Used `ip a` to analyze the network interfaces inside the VM.
2. Updated the `external_url` in the Ansible playbook and `gitlab.rb` to match the actual private network IP.
3. Re-ran `gitlab-ctl reconfigure` to apply the networking changes.

### 🐛 Challenge 2: Ephemeral Containers vs. Artifacts
**The Issue:** In the initial pipeline, the compiled `.jar` file generated in the "Build" stage was disappearing before the "Deploy" stage.
**The Solution:** I learned that Docker containers used by GitLab Runners are destroyed after every job. To fix this, I implemented **GitLab Artifacts** in the `.gitlab-ci.yml` file. This tells GitLab to extract the `target/*.jar` file from the container before it destroys it and store it permanently.

### 🐛 Challenge 3: Port Conflicts
**The Issue:** The default ports for the Puma web server conflicted with other services.
**The Solution:** Modified the `gitlab.rb` configuration to change the Puma port to `8088` and successfully restarted the service using Ansible handlers.

---

## 🚀 How to Run This Project

1.  **Clone the Repository:**
    ```bash
    git clone https://github.com/JabinTasnimUrmy/gitlab-ci-infrastructure-automation.git
    cd gitlab-ci-infrastructure-automation
    ```

2.  **Provision the Server:**
    *(Requires Vagrant and VirtualBox installed)*
    ```bash
    vagrant up
    ```
    *This command will create the VM and automatically run the Ansible provisioning.*

3.  **Access GitLab:**
    *   Open your browser and navigate to `http://192.168.56.9` (or the IP assigned by your Vagrant setup).
    *   Login with the root credentials configured during setup.

4.  **Trigger the Pipeline:**
    *   Push a change to the repository or manually trigger the pipeline in the GitLab UI to see the **Build > Test > Run > Deploy** flow in action.

---

# Building a SOC Lab: Episode 2 – Deploying TheHive, Cortex, and MISP for Security Incident Response

## 🛡️ Introduction

Continuing my journey to build a complete Security Operations Center (SOC) environment, Day 2 focuses on deploying and integrating essential incident response and threat intelligence platforms: TheHive, Cortex, and MISP. These open-source tools form the backbone of any SOC's incident management capabilities.

TheHive is a scalable, open-source security incident response platform designed to make security incidents more manageable. Cortex complements TheHive by providing powerful analysis capabilities for observables, while MISP serves as a threat intelligence platform for sharing and correlating indicators of compromise (IoCs).

## 🧰 Tools Used

For this second day of SOC lab setup, I utilized the following tools:

| Tool | Purpose |
|------|---------|
| **TheHive** | Security Incident Response Platform for case management |
| **Cortex** | Observable analysis engine that integrates with TheHive |
| **MISP** | Malware Information Sharing Platform for threat intelligence |
| **Docker** | Container platform to deploy all services |
| **Docker Compose** | Tool for defining and running multi-container Docker applications |
| **Git** | Version control to clone configuration repositories |
| **Ubuntu Server** | Base operating system for hosting the services |

## 🖥️ Lab Environment Overview

### Servers:
- **Server 1**: Hosting TheHive and Cortex
  - **OS**: Ubuntu 22.04
  - **Specs**: 4 vCPUs, 16GB RAM, 100GB storage
  
- **Server 2**: Hosting MISP
  - **OS**: Ubuntu 22.04
  - **Specs**: 4GB RAM, 80GB storage

### Network Configuration:
- Both servers have static IPs on the same network for integration

## 🛠️ Deployment Procedure

### 1. Deploying TheHive and Cortex (Docker)

For TheHive and Cortex deployment, I followed the official documentation from StrangeBee and used their Docker testing environment, available on [GitHub](https://github.com/StrangeBeeCorp/docker/tree/main/testing).

#### Step 1: Clone the StrangeBee Docker Repository

```bash
git clone https://github.com/StrangeBeeCorp/docker.git
cd docker/testing
```

#### Step 2: Initialize the Environment

The testing environment comes with an initialization script that sets up the necessary configurations:

```bash
bash ./scripts/init.sh
```

This script:
- Prompted for service name for the Nginx certificate
- Initialized the secret.conf files for TheHive and Cortex
- Generated a self-signed certificate
- Created a .env file with user/group information
- Verified file and folder permissions

#### Step 3: Start the Application Stack

```bash
docker compose up -d
```

#### Step 4: Initialize Demo Data and Application Settings (TheHive & Cortex)
The StrangeBee testing environment provides a script to populate demo data and finalize initial application settings, including aspects of TheHive-Cortex integration.

```bash
bash ./scripts/test_init_applications.sh
```
This script automates the creation of demo users, organizations, and ensures Cortex analyzers are activated and configured for TheHive.

### 2. Troubleshooting Elasticsearch Issues

After running `docker compose up -d`, I encountered an issue with the Elasticsearch container showing an unhealthy status.
![Elasticsearch Error](images/thehive/elasticsearch-error.png)

The Elasticsearch container failed to start due to permission issues when writing log files:

```
Exception in thread "main" java.lang.RuntimeException: starting java failed with [1]
output:
[0.000s][error][logging] Error opening log file \'logs/gc.log\': Permission denied
[0.000s][error][logging] Initialization of output \'file=logs/gc.log\' using options \'filecount=32,filesize=64m\' failed.
error:
Invalid -Xlog option \'-Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m\', see error log for details.
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
```

#### The Solution: Using Docker Named Volumes

To fix this issue, I modified the `docker-compose.yml` file to use Docker named volumes instead of host bind mounts for Elasticsearch:

```yaml
# Original configuration with bind mounts
volumes:
  - ./elasticsearch/data:/usr/share/elasticsearch/data
  - ./elasticsearch/logs:/usr/share/elasticsearch/logs

# Updated configuration with named volumes
volumes:
  - es_data:/usr/share/elasticsearch/data
  - es_logs:/usr/share/elasticsearch/logs

# Add at the bottom of docker-compose.yml
volumes:
  es_data:
  es_logs:
```

After making these changes, I restarted the Docker Compose stack:

```bash
docker compose down
docker compose up -d
```
And then re-ran the initialization script for demo data if necessary:
```bash
bash ./scripts/test_init_applications.sh
```

### 3. Accessing TheHive and Cortex

Once all containers were running properly, I could access the web interfaces:

- TheHive: `https://server-ip/thehive`
- Cortex: `https://server-ip/cortex`

You can logging into TheHive with the default credentials (`admin@thehive.local` / `secret`  for admin or `thehive@thehive.local` / `thehive1234` for the default org-admin user) :
![TheHive Login](images/thehive/thehive-login.png)

![TheHive Cases Dashboard](images/thehive/thehive-cases-dashboard.png)

Within TheHive, I also verified the organization and user setup created by the scripts. Also i created a new user `Analyst User` as a test under demo organisation as shown in the image below.

![TheHive Organization List](images/thehive/thehive-org-list.png)

![TheHive User List](images/thehive/thehive-user-list.png)

![TheHive Add User Screen](images/thehive/thehive-add-user.png)

Similarly, I accessed Cortex with the credentials (the default `admin` / `thehive1234` as per `test_init_applications.sh` for the testing environment):
![Cortex Login Page](images/cortex/cortex-login-page.png)

![Cortex Analyzers Page](images/cortex/cortex-analyzers-page.png)

### 4. Deploying MISP on a Separate Server

For MISP deployment, I used the official MISP Docker repository.

#### Step 1: Clone the MISP Docker Repository

```bash
git clone https://github.com/MISP/misp-docker.git
cd misp-docker
```

#### Step 2: Configure Environment Variables

I created a `.env` file based on the example provided:

```bash
cp template.env .env
```

Then edited the file to set the appropriate values:

```
HOSTNAME=misp.local
DOCKER_NETWORK=misp
MYSQL_DATABASE=misp
MYSQL_USER=misp
MYSQL_PASSWORD=misp
MYSQL_ROOT_PASSWORD=misp
MISP_ADMIN_EMAIL=admin@admin.test
MISP_ADMIN_PASSWORD=yourpassword
MISP_BASEURL=https://your-misp-url
```

#### Step 3: Start the MISP Containers

```bash
docker-compose up -d
```

#### Step 4: Initial MISP Setup (Organizations and Users)
After waiting for MISP to initialize (which can take a few minutes), I accessed the web interface using the Base URL we set on the .env file and performed initial setup tasks. This included creating a new organization and a user for interaction with TheHive.

![MISP Login Page](images/misp/misp-login.png)

![MISP Dashboard](images/misp/misp-dashboard.png)

Creating an organization in MISP: To do this go to `Administration` and then select `Add Organisation`

![MISP Create Organisation](images/misp/misp-create-organisation.png)

And viewing the created organization: To view created Organisation go to `View Organisation`
![MISP View Organisation](images/misp/misp-view-organisation.png)

Creating a user in MISP: Follow the same process as creation an organisation  go to `Administration` and then select `Add User` 

![MISP Create User](images/misp/misp-create-user.png)

And viewing the user list:  To view the created User go to `View User`

![MISP View Users](images/misp/misp-view-users.png)

## 🔄 Integration Setup

### 1. Integrating Cortex with TheHive

The `test_init_applications.sh` script from the StrangeBee testing environment automates much of the initial Cortex setup and its integration with TheHive. This includes creating a demo organization in Cortex, a user for TheHive, generating an API key for that user, and pre-configuring TheHive to connect to Cortex.

My steps involved verifying this pre-configuration:

#### Step 1: Verify Cortex Organization and User
In Cortex, I confirmed the existence of the organization (`demo`) and the user (`thehive`) intended for TheHive integration, which should have "read, analyze" roles.

![Cortex Organization Page](images/cortex/cortex-org-page.png)

![Cortex User Page](images/cortex/cortex-user-page.png)

#### Step 2: Verify API Key
The `API key` for the Cortex user is typically generated and configured by the script. I noted where to view/regenerate this key from the `user's profile` in Cortex.

![Cortex User Page with API Key option](images/cortex/cortex-user-page.png)

#### Step 3: Verify TheHive's Cortex Configuration
In TheHive, under Platform Management > Connectors > Cortex, I verified the Cortex server connection details, which should be pre-filled by the script.
- URL: `http://cortex:9001` (as they're in the same Docker network)
- API Key: The pre-configured API key.

![TheHive Cortex Connection](images/thehive/thehive-cortex-connection.png)

A successful connection test confirmed the setup.
![TheHive Cortex Connection Test](images/thehive/thehive-cortex-connection-test.png)

#### Step 4: Ensure Analyzers are Enabled in Cortex
The `test_init_applications.sh` script enables a set of default analyzers. My focus was to ensure the VirusTotal analyzer was specifically configured with my own API key. This involved:

1.  **Obtaining a VirusTotal API Key**: I first created an account on the VirusTotal website and obtained a personal API key.
2.  **Configuring the VirusTotal Analyzer in Cortex**: In Cortex, I navigated to the Administration > Analyzers section, search for the VirusTotal_GetReport and click `+ Enable` and configured it by adding my obtained API key. This ensures that any analysis using VirusTotal would use my account.

![Adding an Analyzer in Cortex](images/cortex/cortex-add-analyzer.png)

![Configuring VirusTotal Analyzer in Cortex](images/cortex/cortex-add-virustotal-analyzer.png)

![VirusTotal Analyzer Enabled in Cortex](images/cortex/cortex-analyzer-view-with-virustotal.png)

### 2. Integrating MISP with TheHive

#### Step 1: Create an API Key in MISP
In MISP, after creating the necessary user (e.g., `labuser@demo.local` as shown in screenshots), I navigated to view that user's profile. From the user's view page, I accessed the `Authkeys` section click the "Add authentication key" button. I then selected the user, optional added a descriptive comment for the key (e.g., "TheHive Integration Key"), set an expiration if desired (or left blank for indefinite), and submitted to generate the key.

![MISP Get Auth Key](images/misp/misp-get-auth-key.png)

The generated key was then visible in the list of authentication keys for that user.

![MISP View Auth Key](images/misp/misp-view-auth-key.png)

#### Step 2: Configure TheHive to Use MISP

In TheHive, I navigated to Platform Management > Connectors > MISP and added a new MISP server:

- URL: `https://misp-server-ip` (IP / domain of the MISP server)
- API Key: Pasted the key generated in MISP 

![TheHive MISP Connection](images/thehive/thehive-misp-connection.png)

With both Cortex and MISP integrated, TheHive's connectors status showed success:
![TheHive MISP and Cortex Success Connections](images/thehive/thehive-misp-cortex-success-connections.png)

## 🔍 Testing the Integration

### 1. Testing Cortex Analysis

To test the Cortex integration, I created a case in TheHive and added an observable (e.g., IP address, hash).
![TheHive Create Case](images/thehive/thehive-create-case.png)

Then I selected the available Cortex analyzers to run on this observable from within TheHive.
![TheHive Showing Cortex Analyzers](images/thehive/thehive-showing-cortex-analyzers.png)

The results were displayed directly in TheHive:
![TheHive Cortex Analysis Report](images/thehive/thehive-cortex-analyisis-report.png)



## 📊 Understanding the SOC Tools

### TheHive: Security Incident Response Platform

TheHive functions as a cybersecurity incident management system—essentially a "case tracker" for security teams. When suspicious activities occur (such as virus alerts, phishing emails, or unauthorized logins), TheHive helps track, organize, and investigate these incidents.

The platform enables team collaboration through:
- Case management
- Task assignment
- Evidence collection
- Notes and documentation
- Observable analysis

![TheHive Cases Dashboard](images/thehive/thehive-cases-dashboard.png)

### Cortex: Observable Analysis Engine

Cortex serves as a cybersecurity toolbox that performs automated analysis. When faced with suspicious elements (files, IP addresses, domains and more), Cortex runs them through various analyzers that check against security databases, providing comprehensive intelligence on potential threats.

Key features include:
- Multiple analyzer integrations (VirusTotal, AbuseIPDB, etc.)
- Automated analysis of observables
- Integration with TheHive for seamless workflow


![Cortex Analyzers Page](images/cortex/cortex-analyzers-page.png)

### MISP: Threat Intelligence Platform

MISP (Malware Information Sharing Platform) functions as a database of indicators of compromise and threat intelligence that security teams can share. It helps teams stay updated on the latest threats by importing and sharing `threat intelligence.`

Key features include:
- Indicator of Compromise (IoC) management
- Threat intelligence sharing
- Automatic correlation of events
- Export capabilities in various formats
- API for integration with other security tools

![MISP Dashboard showing Events](images/misp/misp-dashboard.png)

## 🔄 Workflow Integration

The integration of TheHive, Cortex, and MISP creates a powerful workflow for security incident response:

1.  **Threat Intelligence Ingestion**: MISP serves as a repository for threat intelligence, including Indicators of Compromise (IoCs) from various feeds or shared by other organizations. Alerts or events from MISP can be pulled into TheHive, potentially triggering the creation of new cases or enriching existing ones.
2.  **Case Management & Investigation**: Security analysts use TheHive to manage the lifecycle of an incident. This includes creating cases (either manually or based on MISP alerts), documenting investigation steps, assigning tasks, and collecting evidence (observables).
3.  **Observable Analysis**: During an investigation in TheHive, if analysts encounter suspicious observables (like IP addresses, domains, file hashes, URLs), they can send these directly to Cortex for automated analysis.
4.  **Automated Enrichment**: Cortex, using its array of configured analyzers (including VirusTotal and others, potentially checking against MISP itself if configured), processes the observables. It gathers information such as reputation, known malicious activity, and other relevant details. These analysis results are then sent back to TheHive and automatically appended to the corresponding case and observable, providing analysts with quick, actionable intelligence.
5.  **Correlating with Broader Threat Landscape**: MISP can be queried (either manually or via Cortex analyzers) to check if observables seen in a TheHive case are already known and associated with specific threat actors, malware families, or campaigns, providing broader context to the incident.
6.  **Informed Response**: Based on the comprehensive information gathered in TheHive, enriched by Cortex analysis and contextualized by MISP intelligence, security analysts can make more informed decisions and orchestrate an appropriate response to the incident.

This integrated approach significantly enhances the efficiency and effectiveness of security operations by streamlining case management, automating observable analysis, and leveraging shared threat intelligence.

## 📌 Summary of Achievements

- I successfully deployed TheHive and Cortex using Docker containers from the StrangeBee testing environment.
- Utilized the `test_init_applications.sh` script for initial data and TheHive/Cortex integration.
- Fixed Elasticsearch permission issues using Docker named volumes.
- Deployed MISP on a separate server using Docker.
- Verified TheHive-Cortex integration and enabled necessary analyzers.
- Configured MISP integration with TheHive using a generated API key.
- Tested the workflow from case creation in TheHive to observable analysis with Cortex.
- Successfully established and tested the integrated workflow between TheHive, Cortex, and MISP, gaining practical experience in their combined operational capabilities..

## 🔍 What's Next

For the next part of this series, I plan to:

- Integrating Wazuh SIEM and TheHive so that Wazuh alerts automatically become TheHive alerts/cases for investigation.


Stay tuned for Episode 3! 
# CloudSlang

This repo is dedicated to my attempt to come up to speed on CloudSlang (CS). My thought is, if I can document me coming up to speed, then it may help someone else.

The deployment approach I've chosen for CS is using Docker. This makes running CS stupid easy.

The Dockerfiles can be found in the 'code' section. Also, a README file can be found with each Dockerfile. Those README files contain additional information specific to the Dockerfiles (building, running, verifying that they're working, etc.).

I'm also working on generic use cases with CS content. I've included both my notes on getting these use cases working with CS as well as any CS content (YAML files). Those notes are in README files in the 'content' folder for each use case.


### What is CloudSlang

CS is a free, open source automation engine. This automation engine is based off the commercial enterprise product from Micro Focus (MF) called Operations Orchestration (OO). The use cases for CS center around the same use cases for OO, which is orchestrating any process, large or small, in an agentless manner. More information about CS can be found here:

* The CS home page:
  * <https://cloudslang.io>
* The CS docs:
  * <https://cloudslang-docs.readthedocs.io/en/latest/>
* The CS tutorial:
  * <https://cloudslang-docs.readthedocs.io/en/latest/overview/section_tutorial.html>

    NOTE: I think it works better if you use the 'Minimal' Dockerfile in this repo to run CS in a container, exec into that container, then run through this tutorial.


### How Does CloudSlang Work?

CS works off YAML files. These YAML files define your inputs (variables), run your operations, define your decisions, and outline what navigation to take based on success or failure. This is covered quite well in the tutorial link above.

These YAML files can then be run by CS using one of two methods:

* **Via Command-Line (covered in the tutorial)**

  The CS Command-Line Interface (CLI) can be invoked in one of two methods. You can simply type 'cslang' and enter the CS CLI, similar to Pyhton. After entering the CS CLI interface, you can type 'help' to see a list of CS CLI commands.

  You can also run CS commands without entering the CS CLI by typing 'cslang <cs-command>' from a shell prompt. Typing 'cslang help' from a shell prompt will return the same list of CS CLI commands.
* **Via REST API (not covered in the tutorial)**

  The REST API for CS is actually a completely separate open source project on GitHub called the CS WebApp. The CS WebApp runs an embedded version of CS that exposes CS functionality via a REST based web endpoint.

  Even though the CS WebApp and the CS CLI are completely separate projects, they can be bundled together and run side by side in a single Docker container. That is how the Dockerfiles in this repo are configured. To me, this configuration gives a more complete CS usability story for anyone using CS. Basically, this configuration just stands up a docker container at whatever IP:port, and this container handles all your CS needs. Simple.

  Instructions to test the REST API endpoints in the WebApp are included both in the README for the Dockerfiles in this repo and also on the GitHub page for the cloudslang-webapp project:
  * <https://github.com/CloudSlang/cloudslang-webapp>


### What Can CloudSlang Do?

A better question is "What can't CloudSlang do?"  But more realistically, what can CloudSlang do for you?  I mean, if you're reading this, you probably already have some toolset in place (Jenkins, Ansible, etc.), and some tribal knowledge built around it.  CloudSlang then becomes a "gap filler" between any other products.  For example, here are some smaller, individual processes that could be automated as part of a provisioning/DevSecOps use case:

* Create a ticket in a service management environment (Service Manager, ServiceNow, etc.)
* Provision a VM from template (in vCenter, Proxmox, AWS, Azure, Google Cloud, Oracle Cloud, Alibaba Cloud, etc.)
* Update configurations on a VM
* Trigger a patch update on a VM
* Install application(s), Dev tools, and/or monitoring agent(s) on a VM
* Reboot a VM
* Run Local Tests on a VM (SAST Security Scan, Code Testing, Load Testing, etc.)
* Run External Tests on a VM (DAST Security Scan, App Testing, etc.)
* Collect information from a VM (General health check, Testing results, uptime, Interface Names, IP's, FQDN, Disk Total Size, Disk Space Free, etc.)
* Update and close a ticket in a service management environment
* Email an end user with the completion status

An example of a larger, end-to-end process would be having CS run everything listed above, one after the other, as part of a single automation flow.  The outputs of each step above would feed into the inputs of each step below it. CS can even apply decision making and error handling to provide as much or as little automation to your existing processes as you see fit.  The choice is yours.

The example above centers around a provisioning/DevSecOps use case, but CS can automate all or parts of other use cases (on Linux or Windows, all without agents), including:

* Provisioning Use Cases
  * Retrieve or update IPAM information
  * Provision physical/virtual servers or containers
  * Provision on-prem, cloud, or multi-cloud
* Configuration Management
  * Application Installs
  * Update Config Files, Environment Variables
  * Update Registry Settings
  * Repo/WSUS client configuration
* Patching and Compliance
  * Trigger External Scans (Nessus, ACAS, etc.)
  * Trigger Local Scans & Patching (yum, apt, WSUS, etc.)
* CI/CD & DevSecOps Use Cases
  * Source Management Updates (Git, GitHub, etc.)
  * Trigger Jenkins Integrations
  * Kick-off and verify application tests
  * Trigger SAST / DAST Security Scans
  * Trigger App Testing and/or Load Testing
  * Promote applications/content between environments (DEV -> TEST -> PROD)
* Security Orchestration, Automation, and Responst (SOAR)
  * Have monitoring agents use CS perform their own actions, like:
    * Run security scripts based on specific events
    * Boot unrecognized logins
    * Forcibly close unauthorized port usage
* Service Management Automation Use Cases
  * Update ITSM Environments (Service Manager, ServiceNow, etc.)
    * Create Tickets (Incidents, Change Requests, etc.)
    * Update Tickets (Add info, Change Owners, Move to Queues, etc.)
    * Close Tickets
* Application Lifecycle Management Use Cases
  * Asset Tracking System Updates
* General Business Processes Use Cases
  * New hire onboarding
  * Document Scanning & Data Entry
    * From file share, email, fax systems
    * Into systems with exposed API's


### What's the Catch?

If you're still reading by this point, you have to be thinking there's a "catch" to all this.  There is.  The "catch" is you have to build those automations yourself.  CS provides the content, you have to provide the variables and the logic flow.  A (very) short-list of CS content includes:

* Base Content
  * SSH
    * Runs shell commands or scripts on remote Linux clients
  * PowerShell (WinRM)
    * Runs PowerShell commands or scripts on remote Windows clients
  * Email
    * Send Emails
  * HTTP & HTTPS (for general HTTP/S API's like REST, SOAP, etc.)
    * GET
    * POST
    * PUT
    * DELETE
  * SQL commands
    * Read/write directly to remote databases via SQL
  * Remote File Copy
    * Copy files between remote hosts
  * List, JSON, and XML manipulation
  * And quite a bit more
    * Run the Dockerfile in this repo, exec into the container, then go to '/opt/cslang-cli/content/io/cloudlang' and see for yourself.
* As well as content for specific vendors, like:
  * vCenter
  * Jenkins
  * Ansible
  * Chef
  * Hashicorp
  * OpenShift
  * OpenStack
  * Proxmox
  * Cloud Provider Content
    * AWS
    * Azure
    * Google Cloud
    * Oracle Cloud
    * Digital Ocean
    * Oracle Cloud
  * And again, quite a bit more


### Is the CloudSlang Project Active?

The short answer is yes, the open source project for CS is still active today.

The longer answer is, CS was released as an open source project in 2016. CS is comprised of 2 primary parts:

* **The CS Engine** (that runs the content)
  * <https://github.com/CloudSlang/cloud-slang>
* **The CS Content**
  * <https://marketplace.microfocus.com/ITOM/search?q=cloudslang>
    * Accounts on the MF Marketplace are free to create.

**The CS engine** is based on the same code that runs the enterprise product Operations Orchestration. The CS open source project on GitHub is more of a wrapper for the engine that runs the commercial product. This engine is in a constant commercial development cycle, so the CS/OO engine is constantly updated.

It's also notable that the CS open source project hit many of its usability targets before the end of 2018. Since the CS open source project is more of a wrapper for the CS/OO engine, it doesn't need much more updating. So development of the CS project on GitHub has slowed considerably. But because the CS/OO engine is basically just a mechanism to run the CS content, it's more fair to base activity of the CS project on how current the CS content is.

**The CS content** is constantly being updated and added to. Seriously, click on the link for the CS content above, and you'll see just how active it has been since 2016. The content that runs the free, open source CS engine is also the same content that runs in the enterprise product, Operations Orchestration.

NOTE: The CloudSlang "capsules" on the MF Marketplace are different from the CloudSlang "content." You want the "content" for CloudSlang, not the "capsules." The "capsules" are built off the "content," but include additional integrations for other MF enterprise products (read, not CloudSlang). For CloudSlang, stick to the "content."

I hope this was helpful for anyone exploring CloudSlang.

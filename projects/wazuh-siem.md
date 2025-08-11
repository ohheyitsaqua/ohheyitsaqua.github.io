---
layout: default
title: Wazuh SIEM Deployment
---

# Wazuh SIEM Deployment on AWS EC2

In this project, I deployed Wazuh SIEM/XDR on an Ubuntu EC2 instance, configured agents on Windows and Linux endpoints, and documented the setup process.  
## Wazuh Installation Notes for EC2 Ubuntu
Notes for installing Wazuh SIEM on AWS EC2 Ubuntu instance.

- Part 1:
  - Create AWS EC2 Ubuntu instance with the following settings:
    - Ubuntu EC2 instance (t2.medium is required, higher perferred)
    - At least 4 GB of RAM & 2 vCPU
    - 20GB+ disk storage for logging
    - Security group with inbound access on:
      - TCP 22 for ssh
      - TCP 443 for web UI
      - TCP 1514 for agent logs
      - TCP 1515 for Wazuh registration


 - Personal Notes:
   - Be sure to review the hardware requirements and configuration when setting up the instance. If you neglect to provide enough vCPU, RAM, or Storage the Wazuh installation can fail.
     - Ask me how I know.
     


     
- Part 2:
  - Connect to the instance.
    - Use the following command to connect to the instance. 
      - "ssh -i ~/<key-pair-file.pem> ubuntu@(instancepublicIP)"
        - Note: unless you have a elasticIP provisioned and assigned to the instance, the public IP will change each time so you'll need to update your command each time.

    - You can also configure a ssh config profile to make connecting to the instance easier but as mentioned above, if the publicIP changes each time you'll need to update the config file to match the changed IP.
      - Open or create the config file:
        - Either run this command "nano ~/.ssh/config" or
        - Manually create the file in your .ssh folder.
          - The file needs to be a type "file"; use notepad++ for ease.
      - Add a host block for the ubuntu instance:
        - "Host cyberlab (w/e name you'd like to provide the instance)
            HostName <the EC2 public IP>
            User ubuntu (Default user for Ubuntu AMIs)
            IdentityFile ~/aws-keys/key.pem  # Path to your private key
            IdentitiesOnly yes"
      - Now you can use host ID to ssh into:
        - ex: "ssh cyberlab"

  - Perform updates and upgrades to the instance.
    - Use the following commands to update and upgrade. Additionally, install the git package as well for expanded configuration control.
      - "sudo apt update && sudo apt upgrade -y" - performs update and upgrades system to most recent version.
        - **_personal screenshot removed for security purposes._**

      - "sudo apt install git curl unzip htop net-tools ufw -y" - allows you to use git to manage code on the instance.
        - **_personal screenshot removed for security purposes._**


  - Personal Notes:
    - Don't forget this step to avoid errors down the line.
    - You can also setup an .ssh profile to make the connection easier to command line into.
    - The more you do this, the more you learn the command by motor memory it seems. At least for me.

- Part 3:
  - Install Wazuh on the server.
    - Use the following command to install Wazuh:
      - "curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
        sudo bash wazuh-install.sh -a"
    - You'll be provided with login credentials, be sure to store these somewhere safe (eg: password manager)
      - Access the dashboard insterface using the ip from the server: "https://(wazuh-server-publicIP)"
        - Note this is dynamic same as when starting the server up.
        - **_personal screenshot removed for security purposes._**

   
- Part 4:
  - Install agents on other endpoints to start monitoring logs data to the server.
    - Use the following command(s)/step(s) to install agents:
      - Ubuntu/Linux:
        - "curl -sO https://packages.wazuh.com/4.7/wazuh-agent-4.7.0.deb
           sudo WAZUH_MANAGER='your-wazuh-ec2-ip' dpkg -i ./wazuh-agent-4.7.0.deb
           sudo systemctl enable wazuh-agent
           sudo systemctl start wazuh-agent"
      - Windows:
        - Get the latest Wazuh agent for Windows from the official site: https://packages.wazuh.com/4.x/windows/wazuh-agent-4.x.x.msi
        - Run the installer, leave most of the settings on default.
        - When prompted for the "manager IP" enter the wazuh-ec2-ip.
          - Use port 1514 under the TCP protocol.
          - Provide a name for the agent host.
        - Configure the agent:
          - Find the agent file in the folder: "C:\Program Files (x86)\ossec-agent\ossec.conf" and review the server block. It should look something like this:
            - **_personal screenshot removed for security purposes._**
            - If not, update the block appropriately.
        - Use CMD or Powershell as Admin and run the following command:
          - "sc start WazuhSvc"
          - This should register the agent to the server.
         
  - Personal Notes:
    - When going to install the Ubuntu version of the agent I ran into the following error:
      - **_personal screenshot removed for security purposes._**
      - **_personal screenshot removed for security purposes._**
      - I ran the command twice and received the same error, I'm guessing this is likely due to a corrupted installation.
      - I decided to remove the agent completed and perform a new/clean install:
        - _**personal screenshot removed for security purposes.**_
      - Followed by verifying the installation:
        - **_personal screenshot removed for security purposes._**
      - Clean install:
        - **you get the idea, personal screenshot removed for security purposes.**
      - Success!
    - While this is good practice, I'm likely going to need to assign an elastic IP to the server to ensure everything can connect to it.



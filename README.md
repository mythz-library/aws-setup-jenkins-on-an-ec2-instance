# Setup Jenkins Server on an AWS EC2 Instance

## Table of Contents

- [Setup Jenkins Server on an AWS EC2 Instance](#setup-jenkins-server-on-an-aws-ec2-instance)
  - [Table of Contents](#table-of-contents)
  - [01. Connect to the EC2 Instance](#01-connect-to-the-ec2-instance)
  - [02. Update System Packages](#02-update-system-packages)
  - [03. Install Docker](#03-install-docker)
    - [Set up Docker's apt repository:](#set-up-dockers-apt-repository)
    - [Install the latest Docker packages:](#install-the-latest-docker-packages)
    - [Verify the Docker installation](#verify-the-docker-installation)
  - [04. Configure User Group](#04-configure-user-group)
  - [05. Run Jenkins in a Docker Container](#05-run-jenkins-in-a-docker-container)
  - [06. How to prevent the Jenkins server from stopping when the EC2 instance reboots](#06-how-to-prevent-the-jenkins-server-from-stopping-when-the-ec2-instance-reboots)

<br/><br/>

After configure the AWS EC2 instance, use following steps to configure Jenkins in it.

## 01. Connect to the EC2 Instance

Use SSH to connect to your EC2 instance:

```bash
ssh -i /path/to/your-key.pem ubuntu@<public-ip-of-ec2>
```

<br/>

## 02. Update System Packages

Following command only valid if you're using Ubuntu disto

```bash
sudo apt update && sudo apt upgrade -y
```

<br/>

## 03. Install Docker

[How to install using Docker the `apt` repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)

### Set up Docker's apt repository:

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```bash
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

### Install the latest Docker packages:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Verify the Docker installation

Verify that the installation is successful by running the hello-world image:

```bash
sudo docker run hello-world
```

You have now successfully installed and started Docker Engine.

<br/>

## 04. Configure User Group

Add the **ubuntu** user to the **docker** group to run Docker without `sudo`

```bash
sudo usermod -aG docker ubuntu
```

<br/>

## 05. Run Jenkins in a Docker Container

```bash
# Create a directory for Jenkins data
mkdir -p /var/jenkins_home

# Set proper permissions
sudo chown -R 1000:1000 /var/jenkins_home
```

```bash
# Run the Jenkins container
docker run -d \
  --name jenkins-server \
  -p 8080:8080 \
  -p 50000:50000 \
  -v /var/jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
```

- `-p 8080:8080`: Exposes Jenkins web UI.
- `-p 50000:50000`: For Jenkins agents.
- `-v /var/jenkins_home`: Persists Jenkins data.
- `-v /var/run/docker.sock`: Allows Jenkins to run Docker commands.

<br/>

```bash
# Check the logs to get the initial admin password
docker logs jenkins-server
```

<br/>

## 06. How to prevent the Jenkins server from stopping when the EC2 instance reboots

To automatically restart the Jenkins container instance after EC2 reboot, we can use `--restart unless-stopped` docker's restart policy.

```bash
docker run -d \
  --name jenkins-server \
  --restart unless-stopped \
  -p 8080:8080 \
  -p 50000:50000 \
  -v /var/jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
```

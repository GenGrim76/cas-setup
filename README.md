# Compositional Agile System

## Introduction
Compositional Agile System (CAS) in an open source development environment mainly oriented towards the Agile methodology.\
Included software are:
- [Gitlab](https://about.gitlab.com/), 
- [Taiga](https://www.taiga.io/), 
- [Jenkins](https://www.jenkins.io/), 
- [Sonarqube](https://www.sonarsource.com/products/sonarqube/),
- [Mattermost](https://mattermost.com/).



## Installation

### Prerequisites
- A Debian server with SSH and root access
- A machine with Ansible (may also be the server itself)
  ```
  # To install Ansible
  pip install --user ansible

  # To install Ansible's Docker module
  ansible-galaxy collection install community.docker
  ```

### Settings
The `hosts` file contains some parameters that have to be changed.
- `ansible_host` set to the address of the server
- `ansible_user` set to the user to use during the installation (has to be root or a sudoer)
- `domain` to the domain name or the IP address of the server

#### Enable HTTPS
If `schema` is set to `https`, it will handle the request of a Let's Encrypt certificate (which will automatically accept its [TOS](https://letsencrypt.org/repository/)) and create a cron job for the renewal.\
In order to issue the certificate, the `email` field is required and the server must be publicly reachable on port 80.



### Deployment
```
ansible-playbook main.yml -i hosts
```
If the user requires password authentication, add the `-kK` flag (it will prompt for the password of the user and then for the password to use sudo).

Note: the server's key fingerprint should already be on the Ansible host.



## Initial setup
### Gitlab 
URL: `http(s)://domain/gitlab`\
The `root` user temporary password is located in the container:
```
docker exec cas-gitlab cat /etc/gitlab/initial_root_password
```

### Jenkins 
URL: `http(s)://domain/jenkins`\
The initial password is located in the container:
```
docker exec cas-jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### Sonarqube 
URL: `http(s)://domain/sonarqube`\
The default user is `admin`/`admin`.

Set the `Server base URL` field in **Administration > Configuration > General Settings > General** with the URL of Sonarqube (e.g. http(s)://domain/sonarqube).

### Taiga 
URL: `http(s)://domain/taiga`\
Create an admin user by running:
```
docker exec -it cas-taiga-back python manage.py createsuperuser
```
The admin panel can be found at `http(s)://domain/taiga/admin/` (beware of the final slash).

### Mattermost 
URL: `http(s)://domain/mattermost`\
The first user to signup will automatically become administrator.



## Setup Gitlab SSO
There is an Ansible playbook to setup Gitlab SSO.\
If you prefer to do it manually, some references can be found [here](gitlab-sso.md).

To use the playbook, all services should already have been initialized.\
Run:
```
ansible-playbook sso.yml -i hosts
```
If the user requires password authentication, add the `-kK` flag (it will prompt for the password of the user and then for the password to use sudo).

### Jenkins
Jenkins SSO setup must be done manually. 
1. Install the plugin: **GitLab Authentication**.
2. Go into the **Manage Jenkins > Configure Global Security** tab and select as **Security Realm** `Gitlab Authentication Plugin`.
3. Create a new Gitlab OAuth application (**Admin Area > Applications**) with scope `api` and return URI `/jenkins/securityRealm/finishLogin` (e.g. http(s)://domain/jenkins/securityRealm/finishLogin).
4. On the Jenkins panel insert the required data.


## Add or remove a service
To add a service, create a role and add it to `main.yml`.\
Create a nginx configuration and move it to `{{ nginx_directory }}/includes`.

To remove a service, delete its corresponding row from `main.yml`.


## Useful integrations
- [Gitlab - Taiga](https://docs.taiga.io/integrations-gitlab.html)
- [Gitlab - Mattermost](https://docs.gitlab.com/ee/user/project/integrations/mattermost.html)
- [Gitlab - Jenkins](https://docs.gitlab.com/ee/integration/jenkins.html)
- [Jenkins - Sonarqube](https://docs.sonarqube.org/9.6/analyzing-source-code/scanners/jenkins-extension-sonarqube/)
- [Jenkins - Mattermost](https://www.jenkins.io/doc/pipeline/steps/mattermost/)
- [Discord - Jenkins](https://plugins.jenkins.io/discord-notifier/)
- [Discord - Gitlab](https://docs.gitlab.com/ee/user/project/integrations/discord_notifications.html)

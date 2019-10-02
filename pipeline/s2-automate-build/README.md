# 2- Automate build process


## Assumptions/Pre-requisites

### Software
1. Ansible (v 2.7.5, or higher)
* Instructions to install here: https://docs.ansible.com/
* Check installation with the command `ansible -version`


## Create integration server

The goal is to create a VM that acts as integration server.

At this stage the integration server should be used to hold a VCS where developers 
can publish their changes. Moreover, every time a change is detected the build process 
should be triggered. 

1- Get to the working directory

`cd ~/<git_root_folder>/devops/pipeline/s2-automate-build/integration-server`

2- Create VM
Vagrant is used to create a VM which acts as integration server.

`vagrant up`

**Notes**
The VM is automatically provisioned using Ansible playbooks. 

These playbooks install GitLab and Docker. 

GitLab is used as VCS and CI, whereas Docker is used to handle the integration environments.

**Excerise**
 
* Add package nmap to the playbook.



## Configure GitLab

The goal is to make GitLab accessible as a service.

Note that the installation of GitLab was already made upon starting of the VM (via provisioning).
It only remains to make its configuration.


1- Edit `/etc/gitlab/gitlab.rb` and replace 


`external_url http://hostname`

by

`external_url 'http://192.168.33.9/gitlab' `

and

`# unicorn['port'] = 8080`

by

`unicorn['port'] = 8088`


(Don't forget to uncomment)



2- Reconfigure and restart gitlab

`sudo gitlab-ctl reconfigure`

`sudo gitlab-ctl restart unicorn`

`sudo gitlab-ctl restart`


3- Connect to GitLab

Open `http://192.168.33.9/gitlab` in your browser.

You will be asked to provide a password (refered as $YOUR_PASSWORD later) for the root credentials.

To further login with root the credentials are:<br>
Login: root<br>  
Password: $YOUR_PASSWORD<br>

 
   
**Note:** Upon starting the service up yet another user is requested to be created.



## Configure Docker

The objective of this step is to be able to use the CLI with the current user ($YOUR_USERNAME = vagrant), as the basic
docker daemon and docker CLI have been already installed on the VM upon starting (via provisioning).

1- Add a user to the docker group ot be able to access the docker CLI

`sudo usermod -aG docker $YOUR_USERNAME`


2- Validate the installation and access by running a hello word container.

`docker run --name hello-world hello-world`


The expected output should be.

`Hello from Docker!``

3- Remove the docker container and image:

`docker rm hello-world`

`docker rmi hello-world`




## Use GitLab as VCS

The goal of this step is to make use of GitLab as a VCS. 

1- Create HelloWorldMaven project (in GitLab a project is a repository)

Follow the instructions to create the remote and local repositories.


2- Commit and push to the remote repository.


**Notes:** 

* use the project placed at 
`cd ~/<git_root_folder>/devops/pipeline/s1-create-skeleton/MavenHelloWorldProject`

* ensure the file `pom.xml` is at the root of the local repository. 

* create a file '.gitignore' to avoid committing logs, dev settings, and binaries. 






## Automate build

The objective of this stage is to configure the GitLab project such that
every time a developer publish a change on the remote repository, the build process is 
automatically started. More precisely, upon detection of a change on the remote repository, GitLab
has to:

1. check out/update source code
2. run the automated build script
3. store the binaries where they can be accessed by the team. 



### 5.1-GitLab Runner setup


### 5.2- Create GitLab CI

1- Create file named .gitlab-ci.yml at the root of the repository.

2- Add the following lines into the file:

```
image: maven:latest

stages:
  - build
  - test
  - run
  - deploy


cache:
  paths:
    - target/

build_app:
  stage: build
  script:
    - mvn compile

test_app:
  stage: test
  script:
    - mvn test

run_app:
  stage: run
  script:
    - mvn  package
    - mvn exec:java -Dexec.mainClass="com.jcg.maven.App"

```






   
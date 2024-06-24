Project - Jenkins SonarQube Docker Terraform CI/CD pipeline - Project DONE

This project creates CI/CD Pipeline -> GITHub repro and web hook, docker images for SonarQube, Jenkins and Nginx OnPrem. 

—————————————————————————————————————————————
On-premis docker server (ubuntu 12 core 64 Gb: drop
GIThub Local Repro: /home/jaas/gitcloneWebsite/JenkinsSonarQubeDocker

Docker startup:

docker run -d -it -p 9000:9000 sonarqube1
docker run -d -it -p 8080:8080 -v /home/jaas/jenkins_home:/var/jenkins_home jenkins/jenkins # this will save the config on the local drive
docker run -it --rm -d -p 80:80 -v /home/jaas/gitcloneWebsite/JenkinsSonarQubeDocker:/usr/share/nginx/html nginx

—————————————————————————————————————————————

Create a new GIT hub repository
Copy the website contents from local machine to GIT hub.
Create 3 ec2 instances in aws with ubuntu image
Open Jenkins.oi
	klick on documentation
	klick on insalling Jenkins
	klick on linux /docker
	https://hub.docker.com/r/jenkins/jenkins/


Ssh into the Jenkins server 
$ sudo apt update -y
$ sudo apt install openjdk-11-jre -y

- In aws open SG inbound port for jenkins server tcp 8080:00000/0

- Install Jenkins: 09:00 timestamp (consider using docker image instead then we don’t have to install and configure things)

docker run -d -it -p 8080:8080 jenkins/jenkins

#### Jenkins Pipeline ############################################
- Create Pipeline Jenkins: 
Click new item.
Give it a name like: Auto-pipeline
Click freestyle projet
Click source code management
Select git
Enter repository ULR (copy that from git) and change the name in the branches to build section to Main (as in Github) then click the box for GitHub hook trigger for GITScm polling and then save.
 - This will trigger jenkins when there are updates to the GIT repository.

Click back to Git hub and select settings and web hooks

- Add Webhooks
In the payload box past the jenkins server ip and port and additional text like this: http://13.42.37.188:8080/github-webhook/
Click Let me select individual events.
Select Pull requests, and Pushes
Click the button Add web hook

Verify that the web hook is working by going back to jenkins and in your pipeline , select Build Now, this should complete without errors.
###################################################################

#### Test Jenkins - GitHub WebHook ############################################
Now go to GitHub and add a file in the repository to check that the jenkins pipeline pulls it automatically in the build history.
Time mark: 18:05
------------------------------------------------------------------------

#### SonarQube: ############################################
Log into the sonarQube
Click create a local project
Give the project a display name, like :Nginx-website
Make sure the branch name , is Main (like the git hub repo)
Click next
Click use the global settings, click save/finish
On the page , how do you want to analyse your repository, click Jenkins.
Select GitHub in the DevOps section.
Click on the Other (for js, ts, go, python, php,…) to describe your build.
Copy the sonar.projectkey…………….. to notepad for later use in jenkins.
Click the admin section (top right corner) and select my account
Then click on the security section
Generate a token
Give is a name: sonarqube-token
Select type: either Project analysis or global analysis token
Select project: Nginx-website (from previous config above)
Select expiry: No expiration
Click on generate.
Copy the token, into notepad like before.
#################################################################

Go to jenkins server
Go to dashboard/manage Jenkins/plugins
Click Available plugins
Look for sonarqube scanner and select it to be installed
Click back on Available plugins and look for SSH2 Easy.
Select that and install.

Click manage Jenkins and select Tools under system configuration
Under SonarQube Scanner installation, click Add SonarQube Scanner and give it a name like soanrqube-scanner.
Click save.

Go to manage Jenkins and click system
Under SonarQube servers,  click add SonarQube.
Give it a name: sonar-server.
Past the sonarqube server url in the server url like: http://ip-address:9000
And the click Save 

Click your pipeline and click configure, to add a build step.
Scroll down to build step. Click Add build step and select Execute SonarQube Scanner.
In the analysis properties section, paste the sonar.projectKey…………….
Click Save

Go back to manage Jenkins, system, scroll down to SonarQube servers and add the server authentication token, Jenkins
In add credentials, under Kind, select secret text.
In the secret section , past the secret we copied earlier and give it an ID like : sonar-token.
Click Add
Under SonarQube servers/server authentication token select the newly created toke.
Click Save.

In the pipeline, click |> Build Now to test if it is working. Check the console output for SUCCESS.
(In SonarQube you will have to click around little to make the page update to success.)

############################################################################################
In Dashboard > Manage Jenkins > System, under Server Group Center, Add group list.
Group Name: Docker Server
Check port SSH is : 22
User Name: jaas (for the local onprem ubuntu server)
Password : the password of the root user on the docker server instance (Compaq12..)
Then Click Save.

In Dashboard > Manage Jenkins > System, under Server List, click Add.
Server Group: Docker-Server
Server Name: DockerServer
Server IP: <IP of local ubuntu server running the docker containers> eg 192.168.0.240 

#######################################################################################

In the pipeline, click Configure.
Add build step.
Select docker server (as we are doing to update the local git hub directory with the changes )
Select Remote Shell.
In the Shell section add:
cd /home/jaas/gitcloneWebsite/JenkinsSonarQubeDocker (where the local docker repository is located)
git pull
Click Save

This will pull the changes from GitHub and Pass it through SonarQube Scanner, if there is no issues with the code, Jenkins updates the remote working web directory for NginX and the new content is now available In the web browser.

###################################################################################
Stop docker containers:
log out of SonarQube in browser
 - docker commit boring_curie sonarqube1

Stop all docker containers automatically:
 - docker ps -q | while read i; do docker stop $i; done

Remove all files and folders in repository:
git rm -rf *
git commit -m "remove all"
git push

------------------------------------------------------------------------------

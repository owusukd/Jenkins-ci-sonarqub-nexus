# Jenkins-cicd-sonarqube-ecr-ecs
In this project, I built a java application with maven and uploaded the build to a Nexus server. 
The application is a standalone monolithic java application. 
In this project I provisioned a Jenkins server, SonarQube server for code analysis with defined quality gate and a Nexus server as repo for the build.
I also, created slack notification channel for the project for build notifications. 
The whole process involved in fetching the code from github, performing code anaalysis, building the app, and uploading it to a Nexus server was automated using Jenkins. 

# Build plugins for Jenkins
* Sonarqube
* git
* Nexus
* Pipeline Maven Integration
* BuildTimestamp
* Pipeline utility steps
* Slack notification

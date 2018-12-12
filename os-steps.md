# Steps to Deploy Microservice based application on Openshift Platform

  * Refer - https://github.com/microservices-demo/microservices-demo 
  * It has nearly 10 microservices written in GoLang, SpringBoot, NodeJS
  * Uses MongoDBs, MySQL DBs

# Steps

  * Create 2 projects in Openshift Web Console. One for QA[ms-qa] and another one for Production[ms-prod]
  * oc project ms-qa -  Switch to QA project
  * oc new-app jenkins-persistent -p ENABLE_OAUTH=false -e JENKINS_PASSWORD=admin -n ms-qa - Create Jenkins Instance
  * Increase the Quota to 1.5GB of RAM and 2500 millicores CPU by going Applications -> Deployments -> Jenkins -> Edit Resource Limits
  * Install NodeJS Plugin
  * Go to Global tool configuration and set NodeJS installation path NODE_PATH] by clicking Install Automatically button. Select NodeJS version 9.11.2. Install global package "artillery"
  * oc policy add-role-to-group system:image-puller system:serviceaccounts:ms-prod -n ms-qa - Give permission to PROD to pull the image from QA
  * oc policy add-role-to-user edit system:serviceaccount:ms-qa:jenkins -n ms-prod - Give permission to jenkins to deploy to PROD
  * oc new-app https://github.com/akilans/ms-node-app.git --strategy=docker --name=front-end - Deploy Application on openshift
  * oc expose svc/front-end - Create route for front-end service
  * oc tag front-end:latest front-end:prod - Tag the image for production Deployment
  * oc new-app ms-qa/front-end:prod --name=front-end -n ms-prod
  * oc new-app https://github.com/akilans/ms-node-app.git --strategy=pipeline --name=front-end-pipeline

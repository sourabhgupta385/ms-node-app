# Steps to Deploy Microservice based application on Openshift Platform

  * Refer - https://github.com/microservices-demo/microservices-demo 
  * It has nearly 10 microservices written in GoLang, SpringBoot, NodeJS
  * Uses MongoDBs, MySQL DBs

# Initial steps

  * Create 2 projects in Openshift Web Console. One for QA[ms-qa-sourabh] and another one for Production[ms-prod-sourabh]
  * oc project ms-qa-sourabh -  Switch to QA project
  * oc new-app jenkins-persistent -p ENABLE_OAUTH=false -e JENKINS_PASSWORD=admin -n ms-qa - Create Jenkins Instance
  * Increase the Quota to 1.5GB of RAM and 2500 millicores CPU by going Applications -> Deployments -> Jenkins -> Edit Resource Limits
  * Import https://github.com/sourabhgupta385/openshift-sonarqube/blob/master/sonarqube-ephemeral-template.yaml this yaml to make sonarqube server up
  * Note the sonarqube URL something like: http://sonar-ms-qa-sourabh.apps.na39.openshift.opentlc.com
  * Login in sonarqube server with username as admin and password as admin
  * Generate the token somenthing like cdf76c46cc4be9dc7b60b538266c5fd95b67e7dd and note it
  * Install NodeJS Plugin, SonarQbue Scanner Plugin in jenkins
  * Go to Global tool configuration and set
    * __JDK__ 
      * JDK Name: JAVA_HOME
      * JAVA_HOME: /usr/lib/jvm/java-1.8.0
    * __SonarQube Scanner__
      * Name: SONARQUBE_SCANNER
      * Install automatically with latest version
    * __Maven__
      * Name: MAVEN_HOME
      * Install automatically with latest version
    * __NodeJS__ 
      * Name: NODE_PATH 
      * Install automatically with NodeJS version 9.11.2 
      * Install global package "artillery mocha"
  * Go to Configure system and set
    * __SonarQube Servers__
      * Tick injection of environment variable
      * Name: SonarQube
      * Server URL: Noted in previous steps
      * Secret Token: Noted in previous steps
  * oc policy add-role-to-group system:image-puller system:serviceaccounts:ms-prod-sourabh -n ms-qa-sourabh - Give permission to PROD to pull the image from QA
  * oc policy add-role-to-user edit system:serviceaccount:ms-qa-sourabh:jenkins -n ms-prod-sourabh - Give permission to jenkins to deploy to PROD
  
# Front-End

  * oc new-app https://github.com/sourabhgupta385/ms-node-app.git --strategy=docker --name=front-end - Deploy Application on openshift
  * oc expose svc/front-end - Create route for front-end service
  * oc tag front-end:latest front-end:prod - Tag the image for production Deployment
  * oc new-app ms-qa-sourabh/front-end:prod --name=front-end -n ms-prod-sourabh
  * oc new-app https://github.com/sourabhgupta385/ms-node-app.git --strategy=pipeline --name=front-end-pipeline
  
# Orders

  * oc new-app weaveworksdemos/orders:0.4.7 --name=orders --context-dir=docker/orders
  * oc expose dc/orders --port=80
  * oc tag orders:0.4.7 orders:prod
  * oc new-app ms-qa-sourabh/orders:prod --name=orders -n ms-prod-sourabh
  * oc expose dc/orders --port=80 -n ms-prod-sourabh
  * oc new-build https://github.com/sourabhgupta385/ms-orders.git --strategy=docker --name=orders  [https://docs.openshift.com/container-platform/3.11/dev_guide/dev_tutorials/binary_builds.html ]
  * oc new-app https://github.com/sourabhgupta385/ms-orders.git --strategy=pipeline --name=orders-pipeline
  
# Shipping

  * oc new-app weaveworksdemos/shipping:0.4.8 --name=shipping --context-dir=docker/shipping
  * oc expose dc/shipping --port=80
  * oc tag shipping:0.4.8 shipping:prod
  * oc new-app ms-qa-sourabh/shipping:prod --name=shipping -n ms-prod-sourabh
  * oc expose dc/shipping --port=80 -n ms-prod-sourabh
  * oc new-build https://github.com/sourabhgupta385/ms-shipping.git --strategy=docker --name=shipping [https://docs.openshift.com/container-platform/3.11/dev_guide/dev_tutorials/binary_builds.html ]
  * oc new-app https://github.com/sourabhgupta385/ms-shipping.git --strategy=pipeline --name=shipping-pipeline

# Deploy Remaining services from Image
 
  * oc new-app weaveworksdemos/catalogue:0.3.5 --name=catalogue
  * oc new-app weaveworksdemos/catalogue-db:0.3.0 --env MYSQL_ROOT_PASSWORD=root MYSQL_ALLOW_EMPTY_PASSWORD=true MYSQL_DATABASE=socksdb --name=catalogue-db
  * oc new-app weaveworksdemos/carts:0.4.8 --env JAVA_OPTS="-Xms64m -Xmx128m -XX:+UseG1GC -Djava.security.egd=file:/dev/urandom -Dspring.zipkin.enabled=false" --name=carts
  * oc new-app mongo:3.4 --name=carts-db
  * oc new-app mongo:3.4 --name=orders-db
  * oc new-app weaveworksdemos/queue-master:0.3.1 --name=queue-master
  * oc new-app rabbitmq:3.6.8 --name=rabbitmq
  * oc new-app weaveworksdemos/payment:0.4.3 --name=payment
  * oc new-app weaveworksdemos/user:0.4.4 --env MONGO_HOST=user-db:27017 --name=user
  * oc new-app weaveworksdemos/user-db:0.4.0 --name=user-db
  * oc new-app weaveworksdemos/load-test:0.1.1 --name=user-sim

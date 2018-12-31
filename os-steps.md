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
  
# Orders

  * oc new-app weaveworksdemos/orders:0.4.7 --name=orders --context-dir=docker/orders
  * oc expose dc/orders --port=80
  * oc tag orders:0.4.7 orders:prod
  * oc new-app openshiftplus-dev/orders:prod --name=orders -n openshiftplus-prod
  * oc expose dc/orders --port=80 -n openshiftplus-prod
  * oc new-build . --strategy=docker --name=orders --context-dir=docker/orders [ https://docs.openshift.com/container-platform/3.11/dev_guide/dev_tutorials/binary_builds.html ]
  * oc new-app https://github.com/akilans/ms-orders.git --strategy=pipeline --name=orders-pipeline

# Deploy Remaining services from Image
 
  * oc new-app weaveworksdemos/catalogue:0.3.5 --name=catalogue
  * oc new-app weaveworksdemos/catalogue-db:0.3.0 --env MYSQL_ROOT_PASSWORD=root MYSQL_ALLOW_EMPTY_PASSWORD=true MYSQL_DATABASE=socksdb --name=catalogue-db
  * oc new-app weaveworksdemos/carts:0.4.8 --env JAVA_OPTS="-Xms64m -Xmx128m -XX:+UseG1GC -Djava.security.egd=file:/dev/urandom -Dspring.zipkin.enabled=false" --name=carts
  * oc new-app mongo:3.4 --name=carts-db
  * oc new-app weaveworksdemos/orders:0.4.7 --env JAVA_OPTS="-Xms64m -Xmx128m -XX:+UseG1GC -Djava.security.egd=file:/dev/urandom -Dspring.zipkin.enabled=false" --name=orders
  * oc new-app mongo:3.4 --name=orders-db
  * oc new-app weaveworksdemos/shipping:0.4.8 --env JAVA_OPTS="-Xms64m -Xmx128m -XX:+UseG1GC -Djava.security.egd=file:/dev/urandom -Dspring.zipkin.enabled=false" --name=shipping
  * oc new-app weaveworksdemos/queue-master:0.3.1 --name=queue-master
  * oc new-app rabbitmq:3.6.8 --name=rabbitmq
  * oc new-app weaveworksdemos/payment:0.4.3 --name=payment
  * oc new-app weaveworksdemos/user:0.4.4 --env MONGO_HOST=user-db:27017 --name=user
  * oc new-app weaveworksdemos/user-db:0.4.0 --name=user-db
  * oc new-app weaveworksdemos/load-test:0.1.1 --name=user-sim

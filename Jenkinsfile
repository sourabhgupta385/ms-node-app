node {
   def NODEJS_HOME = tool "NODE_PATH"
   env.PATH="${env.PATH}:${NODEJS_HOME}/bin"
   sh 'npm --version'
   
   stage("Checkout Source"){
       checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/akilans/ms-node-app.git']]])
   }
   
   stage("Install Dependencies"){
        sh 'npm install'
   }
   
   /*
   stage("Code Quality"){
        sh 'npm run lint'
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '', reportFiles: 'quality.html', reportName: 'Quality Report', reportTitles: ''])
   }
   */
   
   stage("Unit Test"){
        sh 'npm run test'
        //publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'coverage', reportFiles: 'index.html', reportName: 'Coverage Report', reportTitles: ''])
   }
   
    stage("Code Coverage"){
        sh 'npm run coverage'
        //publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'coverage', reportFiles: 'index.html', reportName: 'Coverage Report', reportTitles: ''])
   }

   stage("Dev - Building Application"){
        openshiftBuild(buildConfig: 'front-end',showBuildLogs: 'true')
   }

   stage("Dev - Deploying Application"){
       openshiftDeploy(deploymentConfig: 'front-end')
   }
   
   stage("Verify Application"){
       openshiftVerifyService(svcName: 'front-end')
   }
   
   /*
   stage("Functional Testing"){
        sh 'python functionalTest.py'   
   }
   */
      
   stage("Load Testing"){
         sh 'artillery run perfTest.yml'
         //sh 'artillery run perfTest.yml --output load-test.json && artillery report load-test.json --output load-test-result.html'
   }
 
   /*
   stage("Publish Report"){
      
     
      
      publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'functional-test-result', reportFiles: 'index.html', reportName: 'Functional Test report', reportTitles: ''])
      
      publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '', reportFiles: 'load-test-result.html', reportName: 'Load Test report', reportTitles: ''])
   }
   */
   
   
   /*          
   
   stage('Deploy to Production approval'){
      input "Deploy to prod?"
   }
   
   stage("Prod - Building Application"){
        openshiftBuild(namespace:'prod-coe-mern-stack', buildConfig: 'node-backend-app',showBuildLogs: 'true')
   }

   stage("Prod - Deploying Application"){
       openshiftDeploy(namespace:'prod-coe-mern-stack', deploymentConfig: 'node-backend-app')
   }

   */

}

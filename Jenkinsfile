pipeline {
agent any
tools {
maven 'Maven3.6.3'
jdk 'jdk 1.8.211'

}
stages {

stage('StaticCodeAnalysis') {

environment {
scannerhome = tool 'sonarqube'
}

steps {

git 'https://github.com/muthuinit/DevOps-Demo-WebApp.git'

withSonarQubeEnv('sonarqube') {
  sh "${scannerHome}/bin/sonar-scanner -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.projectKey=WEBPOC:AVNCommunication -Dsonar.sources=. -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=admin -Dsonar.password=admin"
}
}

}

stage('BuildWebApp') {
steps {

git 'https://github.com/muthuinit/DevOps-Demo-WebApp.git'

sh "mvn compile"
}
}
stage('DeployToTest') {
steps {

git 'https://github.com/muthuinit/DevOps-Demo-WebApp.git'

sh "mvn package"
}
post {

success {

  deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://104.197.62.1:8080/')],
    contextPath: '/QAWebapp',
    war: '**/*.war'

  echo "'Deploy to QA' - completed successfully."

  //jiraComment body: 'Build Success - Review Build and Deployment Info for Test server', issueKey: 'SQ9DEV-1'
  //jiraSendDeploymentInfo environmentId: 'Test', environmentName: 'Test', environmentType: 'testing', issueKeys: ['SQ9DEV-1'], serviceIds: [''], site: 'hddevopsjiracloud.atlassian.net', state: 'successful'

}
}
}

stage('StoreArtifacts') {

steps {

git 'https://github.com/muthuinit/DevOps-Demo-WebApp.git'

script {

  def server = Artifactory.server 'artifactory'
  def rtMaven = Artifactory.newMavenBuild()
  def buildInfo

  rtMaven.tool = 'Maven3.6.3'

  rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local', server: server

  rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot', server: server

  buildInfo = Artifactory.newBuildInfo()

  buildInfo.env.capture = true

  rtMaven.run pom: 'pom.xml', goals: 'package', buildInfo: buildInfo

  server.publishBuildInfo buildInfo

}
}

}

stage('UITest') {

steps {

git 'https://github.com/muthuinit/DevOps-Demo-WebApp.git'

sh "mvn -f functionaltest/pom.xml test"
}

post {
success {

  publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test', reportTitles: ''])

  echo "UITest results published successfully."
}
}

}

stage('PerformanceTest') {

steps {

blazeMeterTest credentialsId: 'Blazemeter', testId: '747322.taurus', workspaceId: '755428'
}

}

stage('DeployToProd') {
steps {

git 'https://github.com/muthuinit/DevOps-Demo-WebApp.git'

sh "mvn clean install"
}

post {

success {

  deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://35.193.183.11:8080/')],
    contextPath: '/ProdWebapp',
    war: '**/*.war'

  echo "'Deploy to Prod' - completed successfully."

  }
}
}

stage('SanityTest') {

steps {

git 'https://github.com/muthuinit/DevOps-Demo-WebApp.git'

sh "mvn -f Acceptancetest/pom.xml test"
}
post {

always {
  slackSend channel: 'alerts', color: 'good', message: "Started - Job Name - ${env.JOB_NAME}, Build# - ${env.BUILD_NUMBER}, (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'slacknotifications', username: 'jenkins'
}
success {
  publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: ''])

  echo "SanityTest results published successfully."
  slackSend channel: 'alerts', color: 'good', message: "Success - Job Name - ${env.JOB_NAME}, Build# - ${env.BUILD_NUMBER}, (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'slacknotifications', username: 'jenkins'

}

failure {
  slackSend channel: 'alerts', color: 'danger', message: "Failed - Job Name - ${env.JOB_NAME}, Build# - ${env.BUILD_NUMBER}, (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'slacknotifications', username: 'jenkins'
}
}

}
}
}

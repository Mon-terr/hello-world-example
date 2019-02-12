node('master') {
  stage('Poll') {
    checkout scm
  }
  stage('Build & Unit test'){
    withMaven(maven: 'M3') {
      sh 'mvn clean verify -DskipITs=true';
    }
    junit '**/target/surefire-reports/TEST-*.xml'
    archive 'target/*.jar'
  }
  stage ('Publish_Upload Artifactory'){
    def server = Artifactory.server 'Study Artifactory Server'
    def uploadSpec = """{
      "files": [
        {
          "pattern": "target/*.jar",
          "target": "jenkins_test_demo_test/${BUILD_NUMBER}/",
          "props": "Integration-Tested=Yes;Performance-Tested=No"
        }
      ]
    }"""
    server.upload(uploadSpec)
  }
  stash includes: 'target/*.jar',
  name: 'binary'
}
node('master_pt') {

  stage ('Copy File Before Deploy'){
    unstash 'binary'
	sh 'cp target/*.jar /root/nems2/jenkins_deploy_test/';
  }
  
  stage ('Deploy_Production to Servers'){

		sh label: 'deploy_production.sh', script: '/root/nems2/jenkins_deploy_test/deploy_production';
  }
  
  stage('Excution JAR After Deploy') {
	 sh 'date "+%Y-%m-%d %H:%M:%S" >> /root/nems2/jenkins_deploy_test/echo.out';
	 sh 'echo " | This is part of shell script For Test \n" >> /root/nems2/jenkins_deploy_test/echo.out';
  }
}

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
//  stage ('Integration Test'){
//     withMaven(maven: 'M3') {
//        sh 'mvn clean verify -Dsurefire.skip=true';
//      }
//    junit '**/target/failsafe-reports/TEST-*.xml'
//    archive 'target/*.jar'
//  }
  stage ('Publish'){
    def server = Artifactory.server 'Study Artifactory Server'
    def uploadSpec = """{
      "files": [
        {
          "pattern": "target/*.jar",
          "target": "jenkins_test_demo/${BUILD_NUMBER}/",
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
  stage ('Start Tomcat'){
    sh 'cd /root/nems2/deploy_test/';
  }
  stage ('Deploy '){
    unstash 'binary'
    sh 'cp target/*.jar /root/nems2/deploy_test/';
  }
  stage ('Promote build in Artifactory'){
    withCredentials([usernameColonPassword(credentialsId:
     'artifactory-account', variable: 'credentials')]) {
      sh 'curl -u${credentials} -X PUT "http://172.17.201.158:8081/artifactory/api/storage/jenkins_test_demo/${BUILD_NUMBER}/*.jar?properties=Performance-Tested=Yes"';
    }
  }
}

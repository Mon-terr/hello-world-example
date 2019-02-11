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
  stage ('Deploy_SSH transfer'){
    unstash 'binary'
    sh 'cp target/*.jar /root/nems2/jenkins_deploy_test/';
	sshPublisher alwaysPublishFromMaster: true,
	publishers: [
		sshPublisherDesc(
			configName: 'BHMC_deploy',
			sshCredentials: [
				encryptedPassphrase: '{AQAAABAAAAAQLitmD9oMOPIon0LQHqyuaZiJ9ZU3SOyUFEzKYl88FCc=}',
				key: '',
				keyPath: '',
				username: 'root'],
			transfers: [
				sshTransfer(
					cleanRemote: false,
					excludes: '',
					execCommand: 'sh "scp root@172.17.201.158:/root/nems2/jenkins_deploy_test/*.jar /root/nems2/jenkins_deploy_test/"',
					execTimeout: 120000,
					flatten: false,
					makeEmptyDirs: true,
					noDefaultExcludes: false,
					patternSeparator: '[, ]+',
					remoteDirectory: '/jenkins_deploy_test/',
					remoteDirectorySDF: true,
					removePrefix: '/root/nems2/',
					sourceFiles: '/target/*.jar'
				)
			],
			usePromotionTimestamp: false,
			useWorkspaceInPromotion: false,
			verbose: false
		)
	]
	
	
	sshPublisher alwaysPublishFromMaster: true,
	publishers: [
		sshPublisherDesc(
			configName: 'DYK_deploy',
			sshCredentials: [
				encryptedPassphrase: '{AQAAABAAAAAQ17RZCQuR3a0A1lxVM0z8VdmdwQhUmey2IYmXBaUcTSQ=}',
				key: '',
				keyPath: '',
				username: 'root'],
			transfers: [
				sshTransfer(
					cleanRemote: false,
					excludes: '',
					execCommand: 'sh "scp root@172.17.201.158:/root/nems2/jenkins_deploy_test/*.jar /root/nems2/jenkins_deploy_test/"',
					//execCommand: 'echo \'Hello Deploy Success In DYK\' >> /root/nems2/jenkins_deploy_test/echo.out',
					execTimeout: 120000,
					flatten: false,
					makeEmptyDirs: true,
					noDefaultExcludes: false,
					patternSeparator: '[, ]+',
					remoteDirectory: '/jenkins_deploy_test/',
					remoteDirectorySDF: true,
					removePrefix: '/root/nems2/',
					sourceFiles: '/target/*.jar'
				)
			],
			usePromotionTimestamp: false,
			useWorkspaceInPromotion: false,
			verbose: false
		)
	]
  }
  stage('Excution JAR After Deploy') {
	 sh 'date "+%Y-%m-%d %H:%M:%S" >> /root/nems2/jenkins_deploy_test/echo.out';
	 sh 'echo " | This is part of shell script For Test \n" >> /root/nems2/jenkins_deploy_test/echo.out';
  }
}

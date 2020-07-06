pipeline {
    agent {label 'master'}
    stages {
        stage('Cleanup') {
            steps {
                deleteDir()
            }
        }
        stage ('SCM Checkout') {
          steps {
            git (url: "https://github.com/pattubala/JpetStore.git",
            branch: 'master',
            credentialsId: 'Github')
          }
        }
        stage("SONARQUBE") {
            steps {
                script {
				    stage ('Static Code Analysis') {
                        withSonarQubeEnv('Sonarqube_7.6') {
	    			    def SONARQUBE_URL = 'http://40.117.123.236:9000/';
                        sh "mvn sonar:sonar -Dsonar.projectKey=jpetstore -Dsonar.java.binaries=. -Dsonar.language=java -Dsonar.sourceEncoding=UTF-8"
                        } 
				    }
                    stage('Quality Check') {
                        sleep(60);
                        timeout(time: 1, unit: 'MINUTES') { // If something goes wrong pipeline will be killed after a timeout
                        def qg = waitForQualityGate();
                        if (qg.status != "OK") {
                         currentBuild.result='FAILURE';
                         error "Pipeline aborted due to quality gate coverage failure: ${qg.status}"
                        }
                        else{
                           echo "Quality gate passed: ${qg.status}" 
                        }
                        }
                    }	
                }
            }
		}		
		stage ('Maven Build') {
            steps {
                script {
                  sh "mvn clean install"

		        }
		    }
		}
		stage ('Nexus Upload') {
            steps {
                script {
                  nexusPublisher nexusInstanceId: 'Nexus_3.x', nexusRepositoryId: 'Sample_Jpetstore', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: './target/jpetstore.war']], mavenCoordinate: [artifactId: 'maven-jpetstore', groupId: 'maven-org.mybatis', packaging: 'war', version: 'maven-6.0.3-SNAPSHOT']]]

		        }
		    }
		}
		stage ('Email Notification') {
            steps {
                    script {
                        mailRecipients = "chittisreepadh@gmail.com"
                        emailext body: '''$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS: Check console output at $BUILD_URL to view the results.''',
                        mimeType: 'text/html',
                        subject: "${currentBuild.fullDisplayName} - Build ${currentBuild.result}",
                        to: "${mailRecipients}",
                        replyTo: "${mailRecipients}",
                        recipientProviders: [[$class: 'CulpritsRecipientProvider']]

		            }
		    }
		}
    }
}

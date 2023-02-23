def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
pipeline {
	agent any
	tools {
	    maven "MAVEN"
	    jdk "MYJAVA"
	}

	stages {
	    stage('Fetch code') {
            steps {
                git url: 'https://github.com/dexade2010/DevopsBasics.git'
            }

	    }

	    stage('Build'){
	        steps{
	           sh 'mvn install -DskipTests'
	        }

	        post {
	           success {
	              echo 'Now Archiving it...'
	              archiveArtifacts artifacts: '**/target/*.war'
	           }
	        }
	    }

	    stage('UNIT TEST') {
            steps{
                sh 'mvn test'
            }
        }
        
        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR'
            }
        
		steps {
               withSonarQubeEnv('sonar') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=dexade \
                   -Dsonar.projectName=dexade \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=server/src/ \
				   -Dsonar.java.binaries=server/target/classes/com/example/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
			   }
		    }
		}

		stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

		stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: '172.31.27.119:8081/',
                  groupId: 'DEV',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: 'demous',
                  credentialsId: 'nexus-admin',
                  artifacts: [
                    [artifactId: 'demous',
                     classifier: '',
                     file: 'webapp.war',
                     type: 'war']
                    ]
                )
            }
        }
	}
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#groupproject',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
    
}


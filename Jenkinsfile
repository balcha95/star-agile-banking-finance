pipeline {
    agent { label 'slavenode1' }
	

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
	        maven "maven"
    }

	environment {	
		DOCKERHUB_CREDENTIALS=credentials('new_docker_id')
	} 
    stages { 
        stage('SCM Checkout') {
            steps {
                // Get some code from a GitHub repository
                 git 'https://github.com/balcha95/star-agile-banking-finance.git'
            }
		}
        stage('Maven Build') {
            steps {
                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
		}
         stage("Docker build"){
	        steps {
					sh 'docker version'
					sh "docker build -t balcha/banking-app:${BUILD_NUMBER} ."
					sh 'docker image list'
					sh "docker tag balcha/banking-app:${BUILD_NUMBER} balcha/banking-app:latest"
	            }
	        }
	    stage('Login2DockerHub') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'DOCKERHUB_CREDENTIALS', usernameVariable: 'DOCKERHUB_CREDENTIALS_USR', passwordVariable: 'DOCKERHUB_CREDENTIALS_PSW')]) {
            sh """
            echo "${DOCKERHUB_CREDENTIALS_PSW}" > /tmp/dockerhub_password
            docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin < /tmp/dockerhub_password
            rm /tmp/dockerhub_password
            """
        }
    }
}
stage('Login2DockerHub') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}
        stage('Approve - push Image to dockerhub'){
            steps{
                
                //----------------send an approval prompt-------------
                script {
                   env.APPROVED_DEPLOY = input message: 'User input required Choose "yes" | "Abort"'
                       }
                //-----------------end approval prompt------------
            }
        }
		stage('Push2DockerHub') {

			steps {
				sh "docker push balcha/banking-app:latest"
			}
		}
	stage('Deploy to Kubernetes Cluster') {
            steps {
		script {
             sshPublisher(publishers: [sshPublisherDesc(configName: 'kuberneteCluster', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'kubectl apply -f k8sdeployment.yaml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '.', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '*.yaml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])		}
            }
        }
	}
}

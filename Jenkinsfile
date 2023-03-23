pipeline {
    agent { label 'slavenode1' }
    tools {
        maven 'maven'
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
         stage("Docker build") {
	        steps {
					sh 'docker version'
					sh "docker build -t balcha/banking-app:${BUILD_NUMBER} ."
					sh 'docker image list'
					sh "docker tag balcha/banking-app:${BUILD_NUMBER} balcha/banking-app:latest"
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
                     sshPublisher(publishers: [sshPublisherDesc(configName: 'kuberneteCluster', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'kubectl apply -f bankingdeployment.yaml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '.', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '*.yaml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])           
		      }
        	}
        }
    }    
}

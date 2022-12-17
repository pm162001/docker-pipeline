pipeline {
	agent {	
		label 'pipeline-docker'
		}
	stages {
		stage("SCM") {
			steps {
				git branch: 'main', url: 'https://github.com/pm162001/docker-pipeline.git'
				}
			}

		stage("build") {
			steps {
				sh 'sudo mvn dependency:purge-local-repository'
				sh 'sudo mvn clean package'
				}
			}
		stage("Image") {
			steps {
				sh 'sudo docker build -t jenkins-pipeline . '
				sh 'sudo docker tag jenkins-pipeline priyaaa2671/jenkins-pipline '
				}
			}
				
	
		stage("Docker Hub") {
			steps {
			withCredentials([string(credentialsId: 'docker_hub', variable: 'docker_hub_password_var')])   {
				sh 'sudo docker login -u priyaaa2671 -p ${docker_hub_password_var}'
				sh ' sudo docker push priyaaa2671/jenkins-pipeline'
				}
			}
		}

		stage("QAT Testing") {
			steps {
				sh 'sudo docker run -d nginx'
				sh 'sudo docker rm -f $(sudo docker ps -a -q)'
				sh 'sudo docker run -dit -p 8081:8080 --name web1 priyaaa2671/jenkins-pipeline:$BUILD_TAG'
				}
			}
	 	stage("testing website") {
			steps {
				retry(8) {
				sh 'curl --silent http://172.31.8.207:8082/java-web-app/ | grep -i "india" '
				}
	   		}
		}

		stage("Approval status") {
			steps {
				script {  
                		Boolean userInput = input(id: 'Proceed1', message: 'Promote build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
                		echo 'userInput: ' + userInput
				}
	 		}
		}
		
		stage("Prod Env") {
			steps {
			 sshagent(['ec2-call-slave2']) {
			    sh 'ssh -o StrictHostKeyChecking=no ec2-user@3.110.162.80 sudo docker rm -f $(sudo docker ps -a -q)' 
	                    sh "ssh -o StrictHostKeyChecking=no ec2-user@3.110.162.80 sudo docker run  -d  -p  49153:8080  siddharth121/pipeline-java:$BUILD_TAG"
				}
			}
		}
    	}
}

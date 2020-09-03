pipeline {
	agent any
	environment {
		DOCKER_TAG = getDockerTag()
	}
	stages{
		stage('build_war'){
			steps {
				sh  'mvn clean install package'
			}
		}
		stage('lint_checks'){
			steps {
				sh  'hadolint Dockerfile'
			}
		 }
		stage('scan_image'){
			steps {
				aquaMicroscanner imageName: 'tomcat:latest', notCompliesCmd: 'exit 1', onDisallowed: 'fail', outputFormat: 'html'
			}
		}
		stage('docker_build'){
			steps {
				sh 'docker build . -t balavpy20/webapp:${DOCKER_TAG}'
			}
		}
		stage('docker_push'){
		    steps {
			withCredentials([usernamePassword(credentialsId:'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
				sh 'docker login -u ${USERNAME} -p ${PASSWORD}'
				sh 'docker push balavpy20/webapp:${DOCKER_TAG}'
			 }
		    }
       		 }
		
		stage('Deployment'){
			steps {
				sh "aws eks --region us-east-1 update-kubeconfig --name eks-cluster"
				sh "kubectl config set-context --current --namespace=kube-system"
				sh "bash tagscript.sh ${DOCKER_TAG}"
				sh "kubectl apply -f k8-deployment.yml"
				sh "kubectl get nodes"
				sh "kubectl get deployments"
				sh "kubectl get pod -o wide"
				sh "kubectl get services"
				sh "docker image prune -a -f "
			}
		}
		stage('Deployment Status'){
			steps {
				  sh "kubectl rollout status deployments/webapp"
			}
		}
		stage('Application Status'){
			steps {
				sh "chmod +x app_status.sh"
				sh "bash app_status.sh"
			}
		}
	}
}

def getDockerTag () {
	def tag = sh script: 'git rev-parse HEAD', returnStdout:true
	return tag
}

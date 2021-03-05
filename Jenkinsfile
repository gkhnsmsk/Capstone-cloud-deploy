pipeline {
	agent any
	stages {
		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}
		
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker_hub_login', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker build -t gsimsek/cloudcapstone:$BUILD_ID .
					'''
				}
			}
		}

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker_hub_login', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push gsimsek/cloudcapstone:$BUILD_ID
					'''
				}
			}
		}

		stage('Set current kubectl context') {
			steps {
				withAWS(region:'eu-central-1', credentials:'aws_credentials') {
					sh '''
						kubectl config use-context arn:aws:eks:eu-central-1:614257673227:cluster/eksworkshop-eksctl
					'''
				}
			}
		}

		stage('Create blue container') {
			steps {
				withAWS(region:'eu-central-1', credentials:'aws_credentials') {
					sh '''
						kubectl run blueimage --image=gsimsek/cloudcapstone:$BUILD_ID --port=80
					'''
				}
			}
		}

		stage('Expose container') {
			steps {
				withAWS(region:'eu-central-1', credentials:'aws_credentials') {
					sh '''
						kubectl expose deployment blueimage --type=LoadBalancer --port=80
					'''
				}
			}
		}

		stage('Domain redirect blue') {
			steps {
				withAWS(region:'eu-central-1', credentials:'aws_credentials') {
					sh '''
						aws route53 change-resource-record-sets --hosted-zone-id ZKCU19G790VD6 --change-batch file://alias-record.json
					'''
//				}
//			}
//		}
	}
}

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
        withCredentials(bindings: [[$class: 'UsernamePasswordMultiBinding', $CredentialsID':'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]) {
          sh '''
						docker build -t kennethokalc/udacitycapstone .
					'''
        }

      }
    }

    stage('Push Image To Dockerhub') {
      steps {
        withCredentials(bindings: [[$class: 'UsernamePasswordMultiBinding', $CredentialsID:'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]) {
          sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push kennethokalc/udacitycapstone
					'''
        }

      }
    }

    stage('Set current kubectl context') {
      steps {
        withAWS(region: 'us-east-2', credentials: 'capstone_aws_credentials') {
          sh '''
						kubectl config use-context arn:aws:eks:us-east-2:072153795107:cluster/capstonecluster
					'''
        }

      }
    }

    stage('Deploy blue container') {
      steps {
        withAWS(region: 'us-east-2', credentials: 'capstone_aws_credentials') {
          sh '''
						kubectl apply -f ./blue-controller.json
					'''
        }

      }
    }

    stage('Deploy green container') {
      steps {
        withAWS(region: 'us-east-2', credentials: 'capstone_aws_credentials') {
          sh '''
						kubectl apply -f ./green-controller.json
					'''
        }

      }
    }

    stage('Create the service in the cluster, redirect to blue') {
      steps {
        withAWS(region: 'us-east-2', credentials: 'capstone_aws_credentials') {
          sh '''
						kubectl apply -f ./blue-service.json
					'''
        }

      }
    }

    stage('Wait user approve') {
      steps {
        input 'Ready to redirect traffic to green?'
      }
    }

    stage('Create the service in the cluster, redirect to green') {
      steps {
        withAWS(region: 'us-east-2', credentials: 'capstone_aws_credentials') {
          sh '''
						kubectl apply -f ./green-service.json
					'''
        }

      }
    }

  }
}

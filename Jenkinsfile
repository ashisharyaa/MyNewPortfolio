pipeline {
  agent any
  environment {
        DOCKER_IMAGE = 'my-portfolio-web-app'
        DOCKER_TAG = "$DOCKER_Portfolio_IMAGE:${BUILD_ID}"
        CONTAINER_NAME = "portfolio-webapp-${BUILD_ID}"
        KUBECONFIG = '/home/ashish/.kube/config'
    }

  stages {
    stage('Build') {
      steps {
        sh 'docker build -t ${DOCKER_IMAGE} .'
        sh 'docker tag ${DOCKER_IMAGE} ${DOCKER_TAG}'
      }
    }
    stage('Test') {
      steps {
        sh 'docker run --name ${CONTAINER_NAME} -d -p 5050:80 ${DOCKER_TAG}'
      }
    }
    stage('pushing image to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
          sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
          sh 'docker push ${DOCKER_TAG}'
        }
      }
    }
   stage('Deploy to Kubernetes') {
      steps {
        withCredentials([file(credentialsId: "${KUBERNETES_CREDENTIALS_ID}", variable: 'KUBECONFIG')]) {
          sh '''
            kubectl set image deployment.apps/webapp-deployment portfolio-webapp=$DOCKER_Portfolio_IMAGE:${BUILD_ID} --namespace portfolio --kubeconfig $KUBECONFIG
            kubectl rollout status deployment.apps/webapp-deployment --namespace portfolio --kubeconfig $KUBECONFIG
          '''
        }
      }
    }
  }
  post {
    always {
      sh 'docker logout'
    }
  }
}

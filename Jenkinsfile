pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        sh 'docker build -t my-portfolio-web-app .'
        sh 'docker tag my-portfolio-web-app $DOCKER_Portfolio_IMAGE'
      }
    }
    stage('Test') {
      steps {
        sh 'docker run --name portfolio-cont-apps -d $DOCKER_Portfolio_IMAGE'
      }
    }
    stage('pushing image to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
          sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
          sh 'docker push $DOCKER_Portfolio_IMAGE'
        }
      }
    }
   stage('Deploy to Kubernetes') {
      steps {
        withCredentials([file(credentialsId: "${KUBERNETES_CREDENTIALS_ID}", variable: 'KUBECONFIG')]) {
          sh '''
            kubectl set image deployment.apps/webapp-deployment portfolio-webapp=$DOCKER_Portfolio_IMAGE --namespace portfolio --kubeconfig $KUBECONFIG
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

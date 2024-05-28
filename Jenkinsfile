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
        sh 'docker run --name my-portfolio-app-container -d $DOCKER_Portfolio_IMAGE'
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
            kubectl set image deployment/my-portfolio-deployment my-portfolio-container=$DOCKER_Portfolio_IMAGE --kubeconfig $KUBECONFIG
            kubectl rollout status deployment/my-portfolio-deployment --kubeconfig $KUBECONFIG
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

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
        sh 'docker run --name my-portfolio -d -p 8090:80 $DOCKER_Portfolio_IMAGE'
      }
    }
    stage('Deploy') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
          sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
          sh 'docker push $DOCKER_Portfolio_IMAGE'
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
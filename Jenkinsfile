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
        sh 'docker run --name my-portfolio-app-cont -d -p 8070:80 $DOCKER_Portfolio_IMAGE'
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
    stage('Deploying App to Kubernetes') {
      steps {
        script {
          kubernetesDeploy(configs: "deploymentservice.yml", kubeconfigId: "kubernetes")
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

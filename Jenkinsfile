pipeline {
  agent any
  environment {
        DOCKER_IMAGE = 'my-portfolio-web-app'
        DOCKER_TAG = "$DOCKER_Portfolio_IMAGE:${BUILD_ID}"
        CONTAINER_NAME = "portfolio-webapp-${BUILD_ID}"
	KUBECONFIG = "${env.WORKSPACE}/.kube/config"
    }

  stages {
    
    stage('Checkout') {
            steps {
                // Checkout the repository containing the deploymentservice.yml file
                git branch: 'k8s_deploy_feature_branch', url: 'https://github.com/ashisharyaa/MyNewPortfolio.git' // Update with your repository URL
            }
        }

    stage('Build') {
      steps {
        sh 'docker build --no-cache -t ${DOCKER_IMAGE} .'
        sh 'docker tag ${DOCKER_IMAGE} ${DOCKER_TAG}'
        }
      }
    stage('Test') {
      steps {
        sh 'docker run --name ${CONTAINER_NAME} -d ${DOCKER_TAG}'
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
        withCredentials([file(credentialsId: "KUBERNETES_CREDENTIALS_ID", variable: 'KUBECONFIG')]) {
          sh '''#!/bin/bash
            mkdir -p ${env.WORKSPACE}/.kube
            minikube kubectl -- config view --flatten > ${KUBECONFIG_PATH}
            kubectl --kubeconfig=${KUBECONFIG_PATH} apply -f deploymentservice.yml
            kubectl set image deployment.apps/webapp-deployment portfolio-webapp=${DOCKER_TAG} --namespace portfolio --kubeconfig=${KUBECONFIG_PATH}
            kubectl rollout status deployment.apps/webapp-deployment --namespace portfolio --kubeconfig=${KUBECONFIG_PATH}
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

pipeline {
  agent any
  environment {
        DOCKER_IMAGE = 'my-portfolio-web-app'
        DOCKER_TAG = "$DOCKER_Portfolio_IMAGE:${BUILD_ID}"
        CONTAINER_NAME = "portfolio-webapp-${BUILD_ID}"
        KUBECONFIG = "${env.WORKSPACE}/.kube/config" // Path to kubeconfig file
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
        sh 'docker build -t ${DOCKER_IMAGE} .'
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
   stage('Deploy to Minikube') {
            steps {
                script {
                  // Create kubeconfig file if it doesn't already exist
                    sh "mkdir -p ${env.WORKSPACE}/.kube"
                    sh "minikube kubectl -- config view --flatten > ${KUBECONFIG}"

                    // Apply the deployment and service configuration from the YAML file
                    sh "kubectl --kubeconfig=${KUBECONFIG} apply --validate=false -f deploymentservice.yml"

                    // Update the image in the existing deployment
                    sh "kubectl --kubeconfig=${KUBECONFIG} set image deployment.apps/webapp-deployment portfolio-webapp=${DOCKER_TAG} --namespace portfolio"

                    // Ensure the deployment has rolled out
                    sh "kubectl --kubeconfig=${KUBECONFIG} rollout status deployment.apps/webapp-deployment --namespace portfolio"
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

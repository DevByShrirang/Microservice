pipeline {
  agent any

  environment {
    DOCKERHUB_USER = 'shrirang451'
    SERVICE_NAME   = 'adservice'           // 👈 change per branch
    CHART_PATH     = './microservice-chart'
    GIT_CREDENTIAL = 'githubtoken'         // 👈 Jenkins GitHub credentials ID
    GIT_BRANCH     = 'main'                // 👈 where your Helm chart lives
  }

  stages {
    stage('Build & Push Docker Image') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
            echo "🚀 Building and pushing image for ${SERVICE_NAME}"
            sh """
              docker build -t ${DOCKERHUB_USER}/${SERVICE_NAME}:${BUILD_NUMBER} .
              docker push ${DOCKERHUB_USER}/${SERVICE_NAME}:${BUILD_NUMBER}
            """
          }
        }
      }
    }

    stage('Update Helm Values for GitOps') {
      steps {
        script {
          echo "📝 Updating image tag in ${SERVICE_NAME}-values.yaml"
          sh """
            sed -i 's|tag:.*|tag: "${BUILD_NUMBER}"|' ${CHART_PATH}/${SERVICE_NAME}-values.yaml
          """
        }
      }
    }

    stage('Commit & Push Updated Helm Chart') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: "${GIT_CREDENTIAL}", usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
            sh """
              git config user.name "Jenkins CI"
              git config user.email "jenkins@example.com"
              git add ${CHART_PATH}/${SERVICE_NAME}-values.yaml
              git commit -m "🤖 Update ${SERVICE_NAME} image tag to ${BUILD_NUMBER}"
              git push https://${GIT_USER}:${GIT_PASS}@github.com/DevByShrirang/Microservice.git HEAD:${GIT_BRANCH}
            """
          }
        }
      }
    }
  }

  post {
    always {
      echo "🧹 Cleaning up Docker images"
      sh "docker image prune -f || true"
    }
  }
}

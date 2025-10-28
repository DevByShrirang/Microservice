pipeline {
  agent any

  environment {
    DOCKERHUB_USER = 'shrirang451'
    SERVICE_NAME = 'adservice'          // 👈 change this per branch
    CHART_PATH = './microservice-chart' // ✅ matches your repo
  }

  stages {
    stage('Build & Push Docker Image') {
      steps {
        script {
          echo "🚀 Building Docker image for ${SERVICE_NAME}"
          sh """
            docker build -t ${DOCKERHUB_USER}/${SERVICE_NAME}:${BUILD_NUMBER} .
            docker push ${DOCKERHUB_USER}/${SERVICE_NAME}:${BUILD_NUMBER}
          """
        }
      }
    }

    stage('Deploy using Helm') {
      steps {
        script {
          echo "📦 Deploying ${SERVICE_NAME} via Helm..."
          sh """
            helm upgrade --install ${SERVICE_NAME} ${CHART_PATH} \
              -f ${CHART_PATH}/${SERVICE_NAME}-values.yaml \
              --set image.repository=${DOCKERHUB_USER}/${SERVICE_NAME} \
              --set image.tag=${BUILD_NUMBER}
          """
        }
      }
    }

    stage('Verify Deployment') {
      steps {
        script {
          echo "🔍 Verifying deployment for ${SERVICE_NAME}"
          sh "kubectl get pods -l app=${SERVICE_NAME} -n default"
        }
      }
    }
  }

  post {
    always {
      echo "🧹 Cleaning up local Docker cache"
      sh "docker image prune -f || true"
    }
  }
}

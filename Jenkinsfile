pipeline {
  agent any

  environment {
    DOCKERHUB_USER = 'shrirang451'
    SERVICE_NAME   = 'adservice'    // 👈 change per branch
    CHART_PATH     = 'microservice-chart'
    GIT_BRANCH     = 'adservice'
  }

  stages {
    stage('Checkout Code') {
      steps {
        script {
          echo "📥 Checking out ${GIT_BRANCH} branch"
          checkout scm
        }
      }
    }

    stage('Fetch Helm Chart from Main Branch') {
      steps {
        script {
          echo "📦 Fetching ${CHART_PATH} from main branch"
          sh """
            git fetch origin main
            git checkout origin/main -- ${CHART_PATH}
          """
        }
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
            echo "🚀 Building and pushing Docker image for ${SERVICE_NAME}"
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
          echo "📝 Updating image tag in ${CHART_PATH}/${SERVICE_NAME}-values.yaml"
          sh """
            sed -i 's|repository:.*|repository: "${DOCKERHUB_USER}/${SERVICE_NAME}"|' ${CHART_PATH}/${SERVICE_NAME}-values.yaml
            sed -i 's|tag:.*|tag: "${BUILD_NUMBER}"|' ${CHART_PATH}/${SERVICE_NAME}-values.yaml
          """
        }
      }
    }

    stage('Commit & Push Changes') {
      steps {
        script {
          echo "📤 Committing Helm value changes to ${GIT_BRANCH} branch"
          withCredentials([usernamePassword(credentialsId: 'githubtoken', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
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

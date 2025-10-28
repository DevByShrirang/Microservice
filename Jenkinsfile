pipeline {
  agent any

  environment {
    DOCKERHUB_USER = 'shrirang451'
    SERVICE_NAME   = 'adservice'            // 👈 change per branch/service
    CHART_PATH     = 'microservice-chart'   // ✅ corrected path
    GIT_BRANCH     = 'adservice'            // or use env.BRANCH_NAME in multibranch
  }

  stages {

    stage('Checkout Code') {
      steps {
        script {
          echo "📥 Checking out source code for branch: ${GIT_BRANCH}"
          checkout scm
          sh 'echo "✅ Current workspace:" && pwd && ls -R | grep adservice-values.yaml || true'
        }
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        script {
          echo "🚀 Building and pushing Docker image for ${SERVICE_NAME}"
          withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
            sh """
              echo "Building Docker image..."
              docker build -t ${DOCKERHUB_USER}/${SERVICE_NAME}:${BUILD_NUMBER} .
              echo "Pushing image to DockerHub..."
              docker push ${DOCKERHUB_USER}/${SERVICE_NAME}:${BUILD_NUMBER}
            """
          }
        }
      }
    }

    stage('Update Helm Chart for GitOps') {
      steps {
        script {
          echo "📝 Updating image details in Helm chart for ${SERVICE_NAME}"

          // ✅ Check if values file exists
          sh """
            if [ ! -f ${CHART_PATH}/${SERVICE_NAME}-values.yaml ]; then
              echo "❌ ERROR: ${CHART_PATH}/${SERVICE_NAME}-values.yaml not found!"
              echo "Current directory: $(pwd)"
              echo "Available files:"
              ls -R | grep values.yaml || true
              exit 1
            fi
          """

          // ✅ Update repository and tag in the values.yaml
          sh """
            sed -i 's|repository:.*|repository: ${DOCKERHUB_USER}/${SERVICE_NAME}|' ${CHART_PATH}/${SERVICE_NAME}-values.yaml
            sed -i 's|tag:.*|tag: ${BUILD_NUMBER}|' ${CHART_PATH}/${SERVICE_NAME}-values.yaml
            echo "✅ Updated Helm chart for ${SERVICE_NAME}"
          """
        }
      }
    }

    stage('Commit & Push Helm Changes') {
      steps {
        script {
          echo "📤 Committing Helm values update for ${SERVICE_NAME}"
          withCredentials([usernamePassword(credentialsId: 'githubtoken', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
            sh """
              git config --global user.name "Jenkins CI"
              git config --global user.email "jenkins@example.com"

              git add ${CHART_PATH}/${SERVICE_NAME}-values.yaml
              git commit -m "🤖 Update ${SERVICE_NAME} image tag to ${BUILD_NUMBER}" || echo "No changes to commit"

              echo "🔁 Pushing changes to branch: ${GIT_BRANCH}"
              git push https://${GIT_USER}:${GIT_PASS}@github.com/DevByShrirang/Microservice.git HEAD:${GIT_BRANCH}
            """
          }
        }
      }
    }
  }

  post {
    always {
      echo "🧹 Cleaning up Docker images..."
      sh "docker image prune -f || true"
    }
  }
}

pipeline {
  agent any

  environment {
    DOCKERHUB_USER = 'shrirang451'
    SERVICE_NAME   = 'shippingservice'
    CHART_PATH     = 'microservice-chart'
    GIT_BRANCH     = 'shippingservice'         // current feature branch
    MAIN_BRANCH    = 'main'              // branch where Helm chart exists
  }

  stages {

    stage('Checkout Source') {
      steps {
        script {
          echo "📥 Checking out source code for branch: ${GIT_BRANCH}"
          checkout scm
          sh 'echo "✅ Current workspace: $(pwd)" && ls -R | grep Dockerfile || true'
        }
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        script {
          echo "🚀 Building and pushing Docker image for ${SERVICE_NAME}"
          withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
            sh """
              docker build -t ${DOCKERHUB_USER}/${SERVICE_NAME}:${BUILD_NUMBER} .
              docker push ${DOCKERHUB_USER}/${SERVICE_NAME}:${BUILD_NUMBER}
            """
          }
        }
      }
    }

    stage('Update Helm Chart on Main Branch') {
      steps {
        script {
          echo "📝 Switching to main branch to update Helm chart..."

          // ✅ Fetch and checkout main branch separately
          sh """
            git fetch origin ${MAIN_BRANCH}
            git checkout ${MAIN_BRANCH}
          """

          // ✅ Validate that file exists on main
          sh """
            if [ ! -f ${CHART_PATH}/${SERVICE_NAME}-values.yaml ]; then
              echo "❌ ERROR: ${CHART_PATH}/${SERVICE_NAME}-values.yaml not found on main branch!"
              exit 1
            fi
          """

          // ✅ Update image details in Helm values file
          sh """
            sed -i 's|repository:.*|repository: ${DOCKERHUB_USER}/${SERVICE_NAME}|' ${CHART_PATH}/${SERVICE_NAME}-values.yaml
            sed -i 's|tag:.*|tag: ${BUILD_NUMBER}|' ${CHART_PATH}/${SERVICE_NAME}-values.yaml
            echo "✅ Updated Helm chart for ${SERVICE_NAME}"
          """

          // ✅ Commit and push changes to main
          withCredentials([usernamePassword(credentialsId: 'githubtoken', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
            sh """
              git config user.name "Jenkins CI"
              git config user.email "jenkins@example.com"
              git add ${CHART_PATH}/${SERVICE_NAME}-values.yaml
              git commit -m "🤖 Update ${SERVICE_NAME} image tag to ${BUILD_NUMBER}" || echo "No changes to commit"
              echo "📤 Pushing Helm changes to main branch..."
              git push https://${GIT_USER}:${GIT_PASS}@github.com/DevByShrirang/Microservice.git HEAD:${MAIN_BRANCH}
            """
          }

          // ✅ Switch back to service branch
          sh """
            git checkout ${GIT_BRANCH}
          """
        }
      }
    }
  }

  post {
    always {
      echo "🧹 Cleaning up local Docker images..."
      sh "docker image prune -f || true"
    }
  }
}
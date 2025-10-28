pipeline {
  agent any
  environment {
    DOCKERHUB_USER = 'shrirang451'
    SERVICE_NAME = 'adservice'   // 👈 change per branch
    CHART_PATH = './helm/microservice-chart'
    VALUES_PATH = './helm/values'
  }

  stages {
    stage('Build & Push Docker Image') {
      steps {
        sh """
          docker build -t ${DOCKERHUB_USER}/${SERVICE_NAME}:${BUILD_NUMBER} .
          docker push ${DOCKERHUB_USER}/${SERVICE_NAME}:${BUILD_NUMBER}
        """
      }
    }

    stage('Deploy using Helm') {
      steps {
        sh """
          helm upgrade --install ${SERVICE_NAME} ${CHART_PATH} \
            -f ${VALUES_PATH}/${SERVICE_NAME}-values.yaml \
            --set image.repository=${DOCKERHUB_USER}/${SERVICE_NAME} \
            --set image.tag=${BUILD_NUMBER}
        """
      }
    }

    stage('Verify Deployment') {
      steps {
        sh "kubectl get pods -l app=${SERVICE_NAME} -n default"
      }
    }
  }
}

pipeline { 
    agent any

    stages {
        stage('Build & Tag Docker Images to') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t shrirang451/shippingservice:latest ."
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push shrirang451/shippingservice:latest "
                    }
                }
            }
        }
    }
}

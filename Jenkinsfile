pipeline {
    agent any

    environment {
        IMAGE_NAME    = "myapp"
        NEW_VERSION   = "v${BUILD_NUMBER}"
        REGISTRY      = "runtimeterror01"
        GITHUB_REPO   = "https://github.com/RuntimeTerror001/VLE8.git"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: "${GITHUB_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${NEW_VERSION} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
withDockerRegistry([credentialsId: 'docker-creds', url: '']) {
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${NEW_VERSION}"
                }
            }
        }

        stage('Deploy to Green') {
            steps {
                sh """
                  kubectl set image deployment/myapp-green \
                    myapp=${REGISTRY}/${IMAGE_NAME}:${NEW_VERSION}
                  kubectl rollout status deployment/myapp-green
                """
            }
        }

        stage('Health Check') {
            steps {
                sh """
                  sleep 15
                  kubectl get pods -l version=green
                  kubectl rollout status deployment/myapp-green --timeout=60s
                """
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                sh """
                  kubectl patch service myapp-service \
                    -p '{"spec":{"selector":{"version":"green"}}}'
                """
                echo "SUCCESS: Traffic switched to GREEN!"
            }
        }

        stage('Scale Down Blue') {
            steps {
                sh "kubectl scale deployment/myapp-blue --replicas=0"
                echo "Blue environment scaled down. Ready for rollback if needed."
            }
        }
    }

    post {
        failure {
            echo "FAILED: Rolling back to Blue environment!"
            sh """
              kubectl patch service myapp-service \
                -p '{"spec":{"selector":{"version":"blue"}}}'
              kubectl scale deployment/myapp-blue --replicas=2
            """
        }
        success {
            echo "Blue-Green Deployment SUCCESSFUL by RuntimeTerror001!"
        }
    }
}
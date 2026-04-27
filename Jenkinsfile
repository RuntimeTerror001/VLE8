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

        stage('Build Docker Images') {
            steps {
                bat "docker build -t %REGISTRY%/%IMAGE_NAME%:blue-%NEW_VERSION% -f Dockerfile ."
                bat "docker build -t %REGISTRY%/%IMAGE_NAME%:green-%NEW_VERSION% -f Dockerfile.green ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-creds', url: '']) {
                    bat "docker push %REGISTRY%/%IMAGE_NAME%:blue-%NEW_VERSION%"
                    bat "docker push %REGISTRY%/%IMAGE_NAME%:green-%NEW_VERSION%"
                }
            }
        }

        stage('Deploy to Green') {
            steps {
                bat "kubectl set image deployment/myapp-green myapp=%REGISTRY%/%IMAGE_NAME%:green-%NEW_VERSION%"
                bat "kubectl rollout status deployment/myapp-green"
            }
        }

        stage('Health Check') {
            steps {
                bat "kubectl get pods -l version=green"
                bat "kubectl rollout status deployment/myapp-green --timeout=60s"
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                bat "kubectl patch service myapp-service -p \"{\\\"spec\\\":{\\\"selector\\\":{\\\"version\\\":\\\"green\\\"}}}\""
                echo "SUCCESS: Traffic switched to GREEN!"
            }
        }

        stage('Scale Down Blue') {
            steps {
                bat "kubectl scale deployment/myapp-blue --replicas=0"
                echo "Blue environment scaled down."
            }
        }
    }

    post {
        failure {
            echo "FAILED: Rolling back to Blue environment!"
            bat "kubectl patch service myapp-service -p \"{\\\"spec\\\":{\\\"selector\\\":{\\\"version\\\":\\\"blue\\\"}}}\""
            bat "kubectl scale deployment/myapp-blue --replicas=2"
        }
        success {
            echo "Blue-Green Deployment SUCCESSFUL by RuntimeTerror001!"
        }
    }
}
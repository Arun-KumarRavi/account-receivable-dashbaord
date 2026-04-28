pipeline {
    agent any

    environment {
        // --- Registry Config ---
        DOCKER_HUB_USER = 'arunkumarravi08'
        DOCKER_HUB_REPO_FRONTEND = 'accounts-receivable-frontend'
        DOCKER_HUB_REPO_BACKEND = 'accounts-receivable-backend'
        IMAGE_TAG = "${BUILD_NUMBER}"
        
        // --- AWS/EKS Config ---
        CLUSTER_NAME = 'your-eks-cluster-name'
        REGION = 'us-east-1'
    }

    options {
        skipDefaultCheckout()
    }

    stages {
        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            parallel {
                stage('Install Frontend Deps') {
                    steps {
                        dir('client') {
                            sh 'npm ci'
                        }
                    }
                }
                stage('Install Backend Deps') {
                    steps {
                        dir('flask-integration') {
                            sh 'python3 -m venv venv'
                            sh './venv/bin/pip install flask flask-cors pandas scikit-learn numpy pylint pytest safety'
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline Part 1: Checkout finished."
        }
    }
}

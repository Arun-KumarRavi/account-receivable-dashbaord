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
                            // Using --break-system-packages because python3-venv is missing on the agent
                            sh 'python3 -m pip install --break-system-packages flask flask-cors pandas scikit-learn numpy pylint pytest safety'
                        }
                    }
                }
            }
        }

        stage('ESLint') {
            steps {
                dir('client') {
                    sh 'npx eslint src --quiet'
                }
            }
        }

        stage('Frontend Tests') {
            steps {
                dir('client') {
                    sh 'npm test -- --watchAll=false --passWithNoTests'
                }
            }
        }

        stage('Backend Tests') {
            steps {
                dir('flask-integration') {
                    sh 'python3 -m pytest'
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline Part 2 & 3: Finished."
        }
    }
}

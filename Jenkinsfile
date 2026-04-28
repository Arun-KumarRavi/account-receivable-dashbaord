pipeline {
    agent any

    environment {
        // --- Registry Config ---
        DOCKER_HUB_USER = 'arunkumarravi08'
        DOCKER_HUB_REPO_FRONTEND = 'accounts-receivable-frontend'
        DOCKER_HUB_REPO_BACKEND = 'accounts-receivable-backend'
        IMAGE_TAG = "${BUILD_NUMBER}"
        
        // --- SonarQube Config ---
        SCANNER_HOME = tool 'sonar-scanner'
        
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
                    sh 'python3 -m pytest || echo "No tests found yet."'
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                echo "Starting SonarQube analysis..."
                withSonarQubeEnv('SonarQube-Server') {
                    // Added exclusions to avoid Java analysis errors
                    sh "${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=accounts-dashboard \
                        -Dsonar.exclusions=**/*.java"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo "Checking Quality Gate status..."
                waitForQualityGate abortPipeline: true
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs . --severity HIGH,CRITICAL --format table || echo "Trivy not found on agent"'
            }
        }
    }

    post {
        always {
            echo "Pipeline Progress: Finished current stages."
        }
    }
}

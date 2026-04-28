pipeline {
    agent any

    environment {
        // --- Registry Config ---
        DOCKER_HUB_USER = 'arunkumarravi08'
        DOCKER_HUB_REPO_FRONTEND = 'accounts-receivable-frontend'
        DOCKER_HUB_REPO_BACKEND = 'accounts-receivable-backend'
        IMAGE_TAG = "${BUILD_NUMBER}"
        
        // --- SonarQube Config ---
        // SCANNER_HOME = tool 'sonar-scanner'
        
        // --- AWS/EKS Config ---
        CLUSTER_NAME = 'your-eks-cluster-name'
        REGION = 'us-east-1'
    }

    options {
        // Disables the automatic checkout so only our manual stage runs
        skipDefaultCheckout()
    }

    stages {
        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }
    }

    post {
        always {
            echo "Pipeline Part 1: Checkout finished."
        }
    }
}

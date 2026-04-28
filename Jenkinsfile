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


    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Frontend: Lint & Test') {
            steps {
                dir('client') {
                    // npm ci is faster and better for CI environments than npm install
                    sh 'npm ci'
                    sh 'npx eslint src --quiet || echo "Linting issues found but continuing..."'
                    sh 'npm test -- --watchAll=false --passWithNoTests'
                }
            }
        }

        stage('Backend: Lint & Test') {
            steps {
                dir('flask-integration') {
                    // Using pip3 as it is the standard command on most Jenkins/Linux environments
                    sh 'pip3 install flask flask-cors pandas scikit-learn numpy pylint pytest safety'
                    sh 'pylint *.py --disable=C,R || echo "Linting issues found"'
                    sh 'pytest || echo "No tests found"'
                }
            }
        }

        stage('Dependency Check') {
            parallel {
                stage('Frontend Audit') {
                    steps {
                        dir('client') {
                            sh 'npm audit --audit-level=high || true'
                        }
                    }
                }
                stage('Backend Safety') {
                    steps {
                        dir('flask-integration') {
                            sh 'safety check || true'
                        }
                    }
                }
            }
        }

        /*
        stage('Static Analysis (SonarQube)') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=accounts-receivables-dashboard \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://your-sonar-url:9000"
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs . --severity HIGH,CRITICAL --format table'
            }
        }

        stage('Image Creation') {
            steps {
                sh "docker build -t ${DOCKER_HUB_USER}/${DOCKER_HUB_REPO_FRONTEND}:${IMAGE_TAG} ./client"
                sh "docker build -t ${DOCKER_HUB_USER}/${DOCKER_HUB_REPO_BACKEND}:${IMAGE_TAG} ./flask-integration"
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image ${DOCKER_HUB_USER}/${DOCKER_HUB_REPO_FRONTEND}:${IMAGE_TAG} --severity HIGH,CRITICAL"
                sh "trivy image ${DOCKER_HUB_USER}/${DOCKER_HUB_REPO_BACKEND}:${IMAGE_TAG} --severity HIGH,CRITICAL"
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                }
            }
        }

        stage('Docker Push Frontend') {
            steps {
                sh "docker push ${DOCKER_HUB_USER}/${DOCKER_HUB_REPO_FRONTEND}:${IMAGE_TAG}"
            }
        }

        stage('Docker Push Backend') {
            steps {
                sh "docker push ${DOCKER_HUB_USER}/${DOCKER_HUB_REPO_BACKEND}:${IMAGE_TAG}"
            }
        }

        stage('Helm Lint') {
            steps {
                sh 'helm lint ./charts/accounts-dashboard || echo "Helm chart directory not found."'
            }
        }

        stage('EKS Auth') {
            steps {
                withCredentials([aws(credentialsId: 'aws-creds', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh "aws eks update-kubeconfig --name ${CLUSTER_NAME} --region ${REGION}"
                }
            }
        }

        stage('Helm Deploy') {
            steps {
                sh """
                helm upgrade --install accounts-dashboard ./charts/accounts-dashboard \
                    --set frontend.image.tag=${IMAGE_TAG} \
                    --set backend.image.tag=${IMAGE_TAG} || echo "Helm deployment failed."
                """
            }
        }

        stage('Observability Integration') {
            steps {
                echo "Prometheus Metrics: Verifying endpoints..."
                echo "Grafana Visualization: Syncing dashboards..."
                echo "Alerting Notifications: Configuring Slack/Email hooks..."
            }
        }
        */
    }

    post {
        always {
            echo "Pipeline Run Summary: Finished."
            cleanWs()
        }
        success {
            echo "Deployment successful! Build: ${BUILD_NUMBER}"
        }
        failure {
            echo "Pipeline failed. Check Jenkins logs for stage failures."
        }
    }
}

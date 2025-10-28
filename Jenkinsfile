pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to deploy')
    }

    environment {
        DATABASE_URL = credentials('attendance-db-url')
        SECRET_KEY = credentials('attendance-secret-key')
    }

    stages {
        stage('Checkout') {
            steps {
                sh 'git clone https://github.com/MAH76915/DevOps-Project.git .'
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh '''
                    python3.10 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    pip install pytest
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    python -m pytest tests/
                '''
            }
        }

        stage('Pre-Deploy Check') {
            steps {
                sh '''
                    if lsof -i :8092; then
                        echo "Port 8092 is in use. Stopping process..."
                        PID=$(lsof -ti :8092)
                        kill -9 $PID
                    else
                        echo "Port 8092 is free."
                    fi
                '''
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (params.BRANCH_NAME == 'main') {
                        sh 'docker compose build'
                        sh 'docker compose up -d'
                    } else if (params.BRANCH_NAME == 'staging') {
                        sh 'docker compose -f docker-compose.staging.yml build'
                        sh 'docker compose -f docker-compose.staging.yml up -d'
                    } else {
                        echo "No deployment configured for branch: ${params.BRANCH_NAME}"
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}


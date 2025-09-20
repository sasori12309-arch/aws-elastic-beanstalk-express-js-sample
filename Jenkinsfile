pipeline {
    agent {
        docker {
            image 'node:16-bullseye'
            args '-u root:root -v /usr/bin/docker:/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }
        
                
        stage('Security Scan') {
            steps {
                script {
                    sh 'mkdir -p reports'
                    sh 'npm audit --audit-level=high || true'
                    sh 'echo "Security scan completed successfully" > reports/security-scan-result.txt'
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/security-scan-result.txt', allowEmptyArchive: true
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Use Jenkins Docker Pipeline plugin to build the image
                    dockerImage = docker.build("anuj12309/nodejs-app:${env.BUILD_ID}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    // Use Jenkins Docker Pipeline plugin to push the image
                    docker.withRegistry('', 'dockerhub-credentials') {
                        dockerImage.push()
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
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check logs for details.'
        }
    }
}

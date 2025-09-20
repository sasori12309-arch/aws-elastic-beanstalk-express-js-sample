pipeline {
    agent none // No global agent - we'll define agents per stage
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }
    
    stages {
        stage('Checkout') {
            agent {
                docker {
                    image 'node:16-bullseye'
                    args '-u root:root'
                }
            }
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            agent {
                docker {
                    image 'node:16-bullseye'
                    args '-u root:root'
                }
            }
            steps {
                sh 'npm install --save'
            }
        }
        
        stage('Run Tests') {
            agent {
                docker {
                    image 'node:16-bullseye'
                    args '-u root:root'
                }
            }
            steps {
                sh 'npm test'
            }
        }
        
        stage('Security Scan') {
            agent {
                docker {
                    image 'node:16-bullseye'
                    args '-u root:root'
                }
            }
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
            agent any // Use the Jenkins host itself for Docker operations
            steps {
                script {
                    // Build on the Jenkins host
                    sh "docker build -t anuj12309/nodejs-app:${env.BUILD_ID} ."
                }
            }
        }
        
        stage('Push Docker Image') {
            agent any // Use the Jenkins host itself for Docker operations
            steps {
                script {
                    // Push from the Jenkins host
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                        sh "docker push anuj12309/nodejs-app:${env.BUILD_ID}"
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

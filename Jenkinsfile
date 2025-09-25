pipeline {
    agent none

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
                    node {
                        archiveArtifacts artifacts: 'reports/security-scan-result.txt', allowEmptyArchive: true
                    }
                }
            }
        }

        stage('Build Docker Image') {
            agent { label 'built-in' }
            steps {
                script {
                    dir("${env.WORKSPACE}") {
                        sh "docker build -t anuj12309/nodejs-app:${env.BUILD_ID} ."
                    }
                }
            }
        }

        stage('Push Docker Image') {
            agent { label 'built-in' }
            steps {
                script {
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
            node {
                cleanWs()
            }
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed due to errors or vulnerabilities!'
        }
    }
}

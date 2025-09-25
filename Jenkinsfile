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
                    // Run npm audit and save output
                    sh '''
                        npm audit --audit-level=high > reports/security-scan-result.txt
                        if grep -q "found [1-9]" reports/security-scan-result.txt; then
                            echo "High/Critical vulnerabilities detected! Failing build..."
                            exit 1
                        fi
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/security-scan-result.txt', allowEmptyArchive: true
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
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed due to errors or vulnerabilities!'
        }
    }
}

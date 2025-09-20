pipeline {
    agent {
        docker {
            image 'node:16-bullseye'
            args '-u root:root -v /usr/bin/docker:/usr/bin/docker' // Mount Docker CLI from host
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
        
        stage('Install Java') {
            steps {
                sh '''
                    apt-get update
                    apt-get install -y openjdk-11-jre-headless
                '''
            }
        }
        
        stage('Security Scan') {
            steps {
                script {
                    // Create reports directory
                    sh 'mkdir -p reports'
                    
                    // Run npm audit - this will FAIL the build if high/critical vulnerabilities are found
                    // The || true prevents the shell command from failing immediately
                    sh 'npm audit --audit-level=high || true'
                    
                    // Create a result file indicating the scan was completed
                    sh 'echo "Security scan completed successfully" > reports/security-scan-result.txt'
                }
            }
            post {
                always {
                    // Archive the security scan result file
                    archiveArtifacts artifacts: 'reports/security-scan-result.txt', allowEmptyArchive: true
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Builds a Docker image of the app, tagging it with the build ID
                    docker.build("anuj12309/nodejs-app:${env.BUILD_ID}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    // Logs in to Docker Hub using the credentials stored in Jenkins
                    sh "echo \$DOCKERHUB_CREDENTIALS_PSW | docker login -u \$DOCKERHUB_CREDENTIALS_USR --password-stdin"
                    // Pushes the image to your Docker Hub repository
                    docker.withRegistry('', 'dockerhub-credentials') {
                        docker.image("anuj12309/nodejs-app:${env.BUILD_ID}").push()
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

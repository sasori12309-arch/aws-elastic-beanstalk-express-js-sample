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
            // Use npm audit for vulnerability scanning
            try {
                sh 'npm audit --audit-level=high'
                echo 'No high/critical vulnerabilities found'
            } catch (Exception e) {
                error('High or Critical vulnerabilities detected! Build failed.')
            }
        }
    }
    post {
        always {
            // Archive the npm audit log
            archiveArtifacts artifacts: 'npm-debug.log', fingerprint: true
        }
    }
}
      
        
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("anuj12309/nodejs-app:${env.BUILD_ID}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    sh "echo \$DOCKERHUB_CREDENTIALS_PSW | docker login -u \$DOCKERHUB_CREDENTIALS_USR --password-stdin"
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

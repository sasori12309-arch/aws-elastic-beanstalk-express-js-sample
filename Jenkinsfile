pipeline {
    agent {
        docker {
            // Use the custom image with Node.js + Java
            image 'sasori12309-arch/node16-java11'
            args '-u root:root'
        }
    }
    
    environment {
        // Jenkins credentials ID for DockerHub
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
                sh 'npm install'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }
        
        stage('Security Scan') {
            steps {
                sh '''
                mkdir -p ./reports
                wget -q -O dependency-check.zip https://github.com/jeremylong/DependencyCheck/releases/download/v8.2.1/dependency-check-8.2.1-release.zip
                unzip -q dependency-check.zip
                ./dependency-check/bin/dependency-check.sh \
                    --scan . \
                    --format HTML \
                    --out ./reports/dependency-check-report.html \
                    --project "Node.js App"
                '''
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("sasori12309-arch/nodejs-app:${env.BUILD_ID}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    sh "echo \$DOCKERHUB_CREDENTIALS_PSW | docker login -u \$DOCKERHUB_CREDENTIALS_USR --password-stdin"
                    sh "docker push sasori12309-arch/nodejs-app:${env.BUILD_ID}"
                }
            }
        }
    }
    
    post {
        always {
            // Archive Dependency-Check report
            archiveArtifacts artifacts: 'reports/dependency-check-report.html', fingerprint: true
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


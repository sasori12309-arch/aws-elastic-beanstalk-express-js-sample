pipeline {
    agent {
        docker {
            image 'node:18'
            args '-u root:root'
        }
    }
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
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
        
        stage('Install Java') {
            steps {
                sh '''
                    apt-get update
                    apt-get install -y openjdk-17-jre-headless
                '''
            }
        }
        
        stage('Security Scan') {
            steps {
                script {
                    sh 'mkdir -p ./reports'
                    sh 'wget -q -O dependency-check.zip https://github.com/jeremylong/DependencyCheck/releases/download/v8.2.1/dependency-check-8.2.1-release.zip'
                    sh 'unzip -q dependency-check.zip'
                    sh './dependency-check/bin/dependency-check.sh --updateonly || true'
                    sh './dependency-check/bin/dependency-check.sh --scan . --format HTML --out ./reports/dependency-check-report.html --project "Node.js App" --noupdate'
                }
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
            archiveArtifacts artifacts: 'reports/dependency-check-report.html', fingerprint: true
            cleanWs()
        }
    }
}

pipeline {
    agent {
        docker {
            image 'node:18'        // Use Node.js 18 (newer, better supported)
            args '-u root:root'    // Run as root to avoid permission issues
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
                script {
                    // Create reports directory first
                    sh 'mkdir -p ./reports'
                    
                    // Download and install OWASP Dependency-Check
                    sh 'wget -q -O dependency-check.zip https://github.com/jeremylong/DependencyCheck/releases/download/v8.2.1/dependency-check-8.2.1-release.zip'
                    sh 'unzip -q dependency-check.zip'
                    
                    // Try to update database (may fail due to rate limiting)
                    sh './dependency-check/bin/dependency-check.sh --updateonly || true'
                    
                    // Run scan with --noupdate to use local database
                    sh './dependency-check/bin/dependency-check.sh --scan . --format HTML --out ./reports/dependency-check-report.html --project "Node.js App" --noupdate'
                    
                    // Check for vulnerabilities (optional - can remove if you just want the report)
                    def report = readFile('./reports/dependency-check-report.html')
                    if (report.contains('HIGH') || report.contains('CRITICAL')) {
                        error('High or Critical vulnerabilities detected! Build failed.')
                    }
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
            // Archive the Dependency-Check report (Task 4.2 requirement)
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

pipeline {
    agent {
        docker {
            image 'node:16' // Use the standard Node.js 16 image
            args '-u root:root' // Run as root to avoid permission issues
        }
    }
    
    environment {
        // This will reference the credentials you set up in Jenkins
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm // Checks out the code from GitHub
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install --save' // Installs dependencies as required
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'npm test' // Runs the unit tests
            }
        }
        
        stage('Install Java') {
            steps {
                sh '''
                    sed -i 's|http://deb.debian.org/debian|http://archive.debian.org/debian|g' /etc/apt/sources.list
                    sed -i '/security/d' /etc/apt/sources.list
                    apt-get update
                    apt-get install -y openjdk-11-jdk
                '''
            }
        }
        
        stage('Security Scan') {
            steps {
                script {
                    sh 'mkdir -p ./reports'
                    // Download and install OWASP Dependency-Check
                    sh 'wget -q -O dependency-check.zip https://github.com/jeremylong/DependencyCheck/releases/download/v8.2.1/dependency-check-8.2.1-release.zip'
                    sh 'unzip -q dependency-check.zip'
                    // Run the scanner. This generates an HTML report.
                    sh './dependency-check/bin/dependency-check.sh --scan . --format HTML --out ./reports/dependency-check-report.html --project "Node.js App"'
                    
                    // Read the report to check for High/Critical vulnerabilities
                    def report = readFile('./reports/dependency-check-report.html')
                    if (report.contains('HIGH') || report.contains('CRITICAL')) {
                        error('High or Critical vulnerabilities detected! Build failed.') // Fails the pipeline as required
                    }
                }
            }
            post {
                always {
                    // Always archive the security report, even if the build fails
                    archiveArtifacts artifacts: 'reports/dependency-check-report.html', fingerprint: true
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Builds a Docker image of the app, tagging it with the build ID
                    docker.build("sasori12309-arch/nodejs-app:${env.BUILD_ID}")
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
                        docker.image("sasori12309-arch/nodejs-app:${env.BUILD_ID}").push()
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs() // Cleans the workspace after the build
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check logs for details.'
        }
    }
}

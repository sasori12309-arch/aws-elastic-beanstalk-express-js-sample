pipeline {
    agent {
        docker {
            image 'node:16-jdk-bullseye' // Uses Node.js 16 with Java JDK
            args '-u root:root' // Runs as root to avoid file permission issues
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
        
        stage('Security Scan') {
            steps {
                script {
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

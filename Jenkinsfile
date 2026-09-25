pipeline {
    agent any 
    
    options {
        // Keep only the last 5 builds to save storage
        buildDiscarder(logRotator(numToKeepStr: '5')) 
    }

    environment {
        // Use pre-configured Jenkins credentials
        DOCKER_CREDS = credentials('docker-hub-credentials')
        // Defined with your Docker Hub username
        IMAGE_NAME = "zombieli233/nodejs-sample-app"
    }

    stages {
        stage('Install Dependencies & Test') {
            agent {
                // Use Node 16 image as build agent
                docker { 
                    image 'node:16'
                    // Run as root to avoid npm permission issues
                    args '-u root:root' 
                }
            }
            steps {
                echo 'Installing dependencies...'
                sh 'npm install'
                
                echo 'Running unit tests...'
                sh 'npm test || true' 
            }
        }

        stage('Security Scanning') {
            agent {
                docker { 
                    image 'node:16'
                    args '-u root:root'
                }
            }
            steps {
                echo 'Running vulnerability scan...'
                // Fail pipeline if High/Critical vulnerabilities are found
                sh 'npm audit --audit-level=high' 
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                echo 'Building Docker Image...'
                // Build and tag Docker image
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
                
                echo 'Pushing to Docker Registry...'
                // Login and push to Docker Hub
                sh 'echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin'
                sh 'docker push $IMAGE_NAME:$BUILD_NUMBER'
            }
        }
    }

    post {
        always {
            // Archive build artifacts
            archiveArtifacts artifacts: 'package.json, package-lock.json', allowEmptyArchive: true
        }
    }
}

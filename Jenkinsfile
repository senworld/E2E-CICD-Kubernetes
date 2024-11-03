pipeline {
    agent any

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Build') {
            agent {
                docker {
                    image 'maven:3.8.1-openjdk-17'
                }
            }
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'aquasec/trivy:latest'
                    args '--entrypoint=""'
                }
            }
            steps { 
                sh 'ls -l'
                sh 'trivy fs . --cache-dir /tmp/trivy-cache --format table -o fs.html'
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying to environment"
            }
        }
    }
}

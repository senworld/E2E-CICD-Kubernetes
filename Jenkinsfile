pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'maven:3.8.1'
                }
            }
            steps {
                sh 'mvn compile'
                stash includes: './target/*.jar', name: 'compiled-artifact'
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'aquasec/trivy:latest'
                }
            }
            steps {
                unstash 'compiled-artifact'
                echo 'Testing...'
                sh 'ls -l target/*.jar'
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying to environment"
            }
        }
    }
}

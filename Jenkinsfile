pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonarqube-scanner'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Compile') {
            agent {
                docker {
                    image 'maven:3.8.1-openjdk-17'
                }
            }
            steps {
                sh 'mvn compile'
            }
        }
        stage('Trivy Scan FS') {
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
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqubeserver') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Blogging-app -Dsonar.projectKey=Blogging-app \
                          -Dsonar.java.binaries=target'''
                }
            }
        }
        stage('Build') {
            agent {
                docker {
                    image 'maven:3.8.1-openjdk-17'
                }
            }
            steps {
                sh 'mvn package'
            }
        }
        stage('Publish Artifacts') {
            agent {
                docker {
                    image 'maven:3.8.1-openjdk-17'
                }
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                        sh "mvn deploy"
                }
            }
        }
    }
}

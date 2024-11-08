pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonarqube-scanner'
        BUILD_VERSION = "${BUILD_NUMBER.toInteger() / 10}"
    }

    tools {
        maven 'maven'
    }

    stages {

        stage('Compile') {
            steps {
                sh "sed -i '0,/<artifactId>twitter-app<\\/artifactId>/{n; s/<version>.*<\\/version>/<version>${BUILD_VERSION}<\\/version>/}' pom.xml"
                sh 'mvn compile'
            }
        }

        stage('Trivy Scan FS') {
            agent {
                docker {
                    image 'aquasec/trivy:latest'
                    args '--entrypoint=""'
                    reuseNode true
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
            steps {
                sh 'mvn package'
            }
        }

        stage('Publish Artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }

        stage('Docker Build') {
            steps{
                sh "docker build -t 192.168.1.17:8083/repository/docker-repo/twitter-app:${BUILD_VERSION} ."
            }
        }

        stage('Trivy Image Scan') {
            agent {
                docker {
                    image 'aquasec/trivy:latest'
                    args '--entrypoint="" -v /var/run/docker.sock:/var/run/docker.sock'
                    reuseNode true
                }
            }
            steps { 
                sh "trivy image --cache-dir /tmp/trivy-cache --format table -o image.html 192.168.1.17:8083/repository/docker-repo/twitter-app:${BUILD_VERSION}"
            }
        }

        stage('Docker Upload') {
            steps{
                sh "docker push 192.168.1.17:8083/repository/docker-repo/twitter-app:${BUILD_VERSION}"
            }
        }
    }
    post { 
        always { 
            cleanWs()
        }
    }
}   

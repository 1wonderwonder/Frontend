def imageName = "wonderwow/frontend"
def dockerTag = "RC-${env.BUILD_NUMBER}"
def dockerRegistry = ""
def registryCredentials = "dockerhub"

pipeline {
    
    environment {
        PIP_BREAK_SYSTEM_PACKAGES = "1"
        scannerHome = tool 'SonarQube'
    }
    
    agent {
        label 'agent'
    }

    stages {
        stage('Clone repo') {
            steps {
                git branch: 'main', url: 'https://github.com/1wonderwonder/Frontend.git'
            }
        }
        stage('Unit tests') {
            environment {
                PIP_BREAK_SYSTEM_PACKAGES = "1"
            }
            steps {
                sh '''pip3 install -r requirements.txt
                python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml'''
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage('Docker build') {
            steps {
                script {
                    applicationImage =
                    docker.build("${imageName}:${dockerTag}")
                }
            }
        }
        stage("Docker push to Artifactory") {
            steps {
                script {
                    docker.withRegistry("${dockerRegistry}", "${registryCredentials}") {
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }  
            }   
        }
        
    }
    
    post {
      always {
            junit testResults: "test-results/*.xml" 
            cleanWs()
        }
    }
}
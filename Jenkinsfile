pipeline {
    agent {
        label 'agent'
    }
   
    environment {
        scannerHome = tool 'SonarQube'
        PIP_BREAK_SYSTEM_PACKAGES = 1
    }
   
    stages {
        stage('Get Code') {
            steps {
                checkout scm // Get some code from a GitHub repository
            }
        }
        stage('Unit tests') {
            steps {
                sh "pip3 install -r requirements.txt"
                sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml"
            }
        }
        stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage('Build application image') {
            steps {
                script {
                  // Prepare basic image for application
                  dockerTag = "${env.BUILD_ID}.${env.GIT_COMMIT.take(7)}"
                  applicationImage = docker.build("kornzysiek/backend:$dockerTag",".")
                }
            }
        }
        stage ('Pushing image to docker registry') {
            steps {
                script {
                    docker.withRegistry("", "dockerhub") {
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }
            }
        }
    }
    post {
        success {
            build job: 'selenium', 
                  parameters: [
                      string(name: 'backendDockerTag', value: dockerTag)
                  ],
                  wait: false
        }
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }
}
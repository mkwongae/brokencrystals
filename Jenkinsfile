pipeline {
    agent {
        label "built-in"
    }
    environment {
        APP_NAME = "brokencrystals"
        RELEASE = "0.0.71"
        DOCKER_USER = "mkwongae"
        DOCKER_PASS = "dockerhub-jenkins"
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        SEMGREP_APP_TOKEN = credentials('SEMGREP_APP_TOKEN')
        SEMGREP_PR_ID = "${env.CHANGE_ID}"
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }
        stage("Checkout from Git") {
            steps {
                git branch: 'stable', url: 'https://github.com/mkwongae/brokencrystals.git'
            }
        }
        stage('Semgrep Scan') {
            steps {
                script {
                    sh 'pip3 install semgrep'
                    sh 'semgrep ci'
                }
            }
        }
        stage("SonarQube Analysis") {
            steps {
                script {
                    scannerHome = tool 'sonarqube'
                }
                withSonarQubeEnv(installationName: 'sonarqube', credentialsId: 'sonarqube-jenkins') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage("Build and Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image = docker.build("${IMAGE_NAME}")
                    }

                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }
                }
            }
        }
        stage("Trivy Image Scan") {
            steps {
                script {
                    def trivyOutput = sh(script: "trivy image ${docker_image.id}", returnStdout: true).trim()
                    println trivyOutput
                }
            }
        }
    }
}
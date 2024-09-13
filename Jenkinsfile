pipeline {
    agent {
        label "agent1"
    }
    environment {
        APP_NAME = "brokencrystals"
        RELEASE = "0.0.71"
        DOCKER_USER = "mkwongae"
        DOCKER_PASS = "dockerhub-jenkins"
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IAMGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
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
        // stage("SonarQube Analysis") {
        //     steps {
        //         script {
        //             withSonarQubeEnv(credentialsId: 'sonarqube-jenkins') {
                    
        //             }
        //         }
        //     }
        // }
        stage("Build and Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }
                }
            }
        }
    }
}
pipeline {
    agent {
        docker {
            image 'ubuntu:latest'
        }
    }

    parameters {
        string(name: 'IMG_TAG', defaultValue: 'latest', description: 'Add a tag for your Image')
    }

    environment {
        IMG  = 'my_python_app'
        REPO = 'cmuriukin/class_24_repository'
    }

    stages {
        stage('Checkout Git Project') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${env.BRANCH_NAME}"]],
                    userRemoteConfigs: [[url: 'https://github.com/cmuriukin/my-simple-python-application.git']]
                ])
            }
        }

        stage('Docker Build Image') {
            steps {
                sh 'docker build -t $IMG:$IMG_TAG .'
            }
        }

        stage('Scan Docker Image') {
            steps {
                script {
                    def scanStatus = sh(
                        script: 'trivy image --exit-code 1 --severity CRITICAL $IMG:$IMG_TAG',
                        returnStatus: true
                    )

                    if (scanStatus != 0) {
                        slackSend (
                            color: 'warning',
                            message: "Docker Build ${BUILD_NUMBER} on branch ${env.BRANCH_NAME} found CRITICAL vulnerability",
                            channel: '#class_24_docker_project',
                            tokenCredentialId: 'slack_channel'
                        )
                    }
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh 'docker tag $IMG:$IMG_TAG $REPO:$IMG_TAG'
                sh 'docker tag $IMG:$IMG_TAG $REPO:$BUILD_NUMBER'
                sh 'docker tag $IMG:$IMG_TAG $REPO:${env.BRANCH_NAME}-${BUILD_NUMBER}'
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker_credential',
                    passwordVariable: 'DOCKERHUB_PASSWORD',
                    usernameVariable: 'DOCKER_USERNAME'
                )]) {
                    sh "echo $DOCKERHUB_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    sh 'docker push $REPO:$IMG_TAG'
                    sh 'docker push $REPO:$BUILD_NUMBER'
                    sh 'docker push $REPO:${env.BRANCH_NAME}-${BUILD_NUMBER}'
                }
            }
        }
    }

    post {
        success {
            slackSend (
                color: 'good',
                message: "✅ Docker Build ${BUILD_NUMBER} on branch ${env.BRANCH_NAME} completed successfully",
                channel: '#class_24_docker_project',
                tokenCredentialId: 'slack_channel'
            )
        }
        failure {
            slackSend (
                color: 'danger',
                message: "❌ Docker Build ${BUILD_NUMBER} on branch ${env.BRANCH_NAME} failed",
                channel: '#class_24_docker_project',
                tokenCredentialId: 'slack_channel'
            )
        }
    }
}

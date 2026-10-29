pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Add tag for your image')
        string(name: 'IMAGE_NAME', defaultValue: 'simple_image_name', description: 'Add image name for your docker image')
    }

    environment {
        REPO = 'chandra237/class_28_repo'
    }

    stages {
        stage('Checkout Git Repository') {
            steps {
                git branch: 'feature-class_28', url: 'https://github.com/cwule06/my-simple-python-application.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Scan Docker Image') {
            steps {
                sh 'trivy image $IMAGE_NAME:$IMAGE_TAG'
                //sh 'trivy image --exit-code 1 --severity CRITICAL,HIGH $IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh 'docker tag $IMAGE_NAME:$IMAGE_TAG $REPO:$IMAGE_TAG'
                sh 'docker tag $IMAGE_NAME:$IMAGE_TAG $REPO:$BUILD_NUMBER'
            }
        }

        stage('Docker push Image dockerhub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Jenkins_PAT', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USER')]) {
                    sh 'echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USER --password-stdin'
                    sh 'docker push $REPO:$IMAGE_TAG'
                    sh 'docker push $REPO:$BUILD_NUMBER'
                }
            }
        }
    }

    post {
        success {
            slackSend channel: '#all-devops-class',
                color: 'good',
                message: "Docker build Id $BUILD_NUMBER was built and published Successfully",
                tokenCredentialId: 'class_28_slack_jenkins_cred'
        }
        failure {
            slackSend channel: '#all-devops-class',
                color: 'danger',
                message: "Docker pipeline failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                tokenCredentialId: 'class_28_slack_jenkins_cred'
        }
        always {
            sh 'docker logout || true'
        }
    }
}

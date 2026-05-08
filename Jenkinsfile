pipeline {
    agent any

    environment {
        IMAGE_NAME = "mahender397/devops-webapp"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/mahi3297/devops-cicd-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('app') {
                    sh 'docker build -t $IMAGE_NAME:$TAG .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $IMAGE_NAME:$TAG'
                }
            }
        }

        stage('Update Helm Values') {
            steps {
                sh '''
                sed -i "s/tag:.*/tag: \"${TAG}\"/" helm/webapp/values-dev.yaml
                '''
            }
        }

        stage('Git Push Updated Helm') {
            steps {
                sh '''
                git config --global user.email "jenkins@example.com"
                git config --global user.name "jenkins"
                git add .
                git commit -m "Updated image tag"
                git push
                '''
            }
        }
    }
}

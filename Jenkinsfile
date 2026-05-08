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
                credentialsId: 'github-creds',
                url: 'https://github.com/mahi3297/devops-cicd-project.git'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('app') {
                    sh '''
                    docker build -t $IMAGE_NAME:$TAG .
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                docker push $IMAGE_NAME:$TAG
                '''
            }
        }

        stage('Update Helm Values') {
            steps {
                sh '''
                sed -i "s/tag:.*/tag: \\"${TAG}\\"/" helm/webapp/values-dev.yaml
                sed -i "s/tag:.*/tag: \\"${TAG}\\"/" helm/webapp/values-stage.yaml
                sed -i "s/tag:.*/tag: \\"${TAG}\\"/" helm/webapp/values-prod.yaml
                '''
            }
        }

        stage('Git Push Updated Helm') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {

                    sh '''
                    git config --global user.email "jenkins@example.com"
                    git config --global user.name "jenkins"

                    git add .

                    git commit -m "Updated image tag to ${TAG}" || echo "No changes to commit"

                    git push https://${GIT_USER}:${GIT_PASS}@github.com/mahi3297/devops-cicd-project.git HEAD:main
                    '''
                }
            }
        }
    }
}

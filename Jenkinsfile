pipeline {
    agent any

    environment {
        // ===== UPDATE THESE VALUES FOR YOUR ENVIRONMENT =====
        IMAGE_NAME = "mahender397/devops-webapp"
        TAG = "${BUILD_NUMBER}"

        GIT_REPO   = "https://github.com/mahi3297/devops-cicd-project.git"
        GIT_BRANCH = "main"

        // Jenkins credential IDs
        DOCKER_CREDENTIALS = "dockerhub-creds"
        GITHUB_CREDENTIALS = "github-creds"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${GIT_BRANCH}",
                    credentialsId: "${GITHUB_CREDENTIALS}",
                    url: "${GIT_REPO}"
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('app') {
                    sh '''
                        docker build -t ${IMAGE_NAME}:${TAG} .
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push ${IMAGE_NAME}:${TAG}
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

        stage('Commit and Push Helm Changes') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${GITHUB_CREDENTIALS}",
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_TOKEN'
                )]) {
                    sh '''
                        git config --global user.email "jenkins@example.com"
                        git config --global user.name "Jenkins"

                        git add helm/webapp/values-*.yaml

                        # Commit only if there are changes
                        git diff --cached --quiet || \
                        git commit -m "Update image tag to ${TAG}"

                        # Use HTTPS with GitHub PAT for push
                        git remote set-url origin \
                          https://${GIT_USER}:${GIT_TOKEN}@github.com/mahi3297/devops-cicd-project.git

                        git push origin ${GIT_BRANCH}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully."
            echo "Docker Image: ${IMAGE_NAME}:${TAG}"
            echo "ArgoCD will automatically sync dev, stage, and prod."
        }

        failure {
            echo "Pipeline failed. Check the stage logs above."
        }

        always {
            sh 'docker logout || true'
        }
    }
}

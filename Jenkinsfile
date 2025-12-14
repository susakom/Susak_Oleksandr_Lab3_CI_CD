/*
  Jenkinsfile for CICD Multibranch Pipeline

  Requirements covered:
  - Same Jenkinsfile for main and dev
  - NodeJS version via Global Tools Configuration
  - npm install / npm test
  - Different Docker images for main and dev
  - Different ports for main and dev
*/

pipeline {

    agent any

    environment {
        // Base container name (used for stop/remove)
        CONTAINER_NAME = "nodeapp"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checkout source code from branch: ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage('Build') {
            steps {
                nodejs('node18') {
                    echo "Installing NodeJS dependencies"
                    sh 'npm install'
                }
            }
        }

        stage('Test') {
            steps {
                nodejs('node18') {
                    echo "Running tests"
                    sh 'npm test'
                }
            }
        }

        stage('Set Environment Variables') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.IMAGE_NAME = 'nodemain:v1.0'
                        env.APP_PORT  = '3000'
                        echo "Main branch detected"
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.IMAGE_NAME = 'nodedev:v1.0'
                        env.APP_PORT  = '3001'
                        echo "Dev branch detected"
                    } else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image ${env.IMAGE_NAME}"
                sh """
                    docker build -t ${env.IMAGE_NAME} .
                """
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-creds',
                            usernameVariable: 'DOCKERHUB_USER',
                            passwordVariable: 'DOCKERHUB_PASS'
                        )
                    ]) {
                        sh """
                            echo "Login to Docker Hub"
                            echo \$DOCKERHUB_PASS | docker login -u \$DOCKERHUB_USER --password-stdin

                            echo "Tag image for Docker Hub"
                            docker tag ${env.IMAGE_NAME} \$DOCKERHUB_USER/lab3_ci_cd:${env.BRANCH_NAME}

                            echo "Push image to Docker Hub"
                            docker push \$DOCKERHUB_USER/lab3_ci_cd:${env.BRANCH_NAME}

                            docker logout
                        """
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying application on port ${env.APP_PORT}"

                sh """
                    # Stop and remove containers using the same port
                    docker ps --filter "publish=${env.APP_PORT}" -q | xargs -r docker rm -f

                    echo "Removing container with the same name (if exists)..."
                    docker rm -f ${CONTAINER_NAME}-${env.BRANCH_NAME} || true

                    # Run new container with minimal downtime
                    docker run -d \
                        --name ${CONTAINER_NAME}-${env.BRANCH_NAME} \
                        --expose ${env.APP_PORT} \
                        -p ${env.APP_PORT}:3000 \
                        ${env.IMAGE_NAME}
                """
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully for branch ${env.BRANCH_NAME}"
        }
        failure {
            echo "Pipeline failed for branch ${env.BRANCH_NAME}"
        }
    }
}

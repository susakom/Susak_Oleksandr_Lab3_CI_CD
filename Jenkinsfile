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

        stage('Trigger CD Pipeline') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        echo "Triggering Deploy_to_main pipeline"
                        build job: 'Deploy_to_main', wait: false
                    } else if (env.BRANCH_NAME == 'dev') {
                        echo "Triggering Deploy_to_dev pipeline"
                        build job: 'Deploy_to_dev', wait: false
                    }
                }
            }
        }
    }

    post {
        success {
            echo "CI pipeline completed successfully for branch ${env.BRANCH_NAME}"
        }
        failure {
            echo "CI pipeline failed for branch ${env.BRANCH_NAME}"
        }
    }
}

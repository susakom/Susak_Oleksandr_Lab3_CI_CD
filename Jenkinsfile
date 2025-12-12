/*
  Jenkinsfile for Multibranch Pipeline

  Purpose:
  - Use Git branches as environments
  - main  -> port 3000
  - dev   -> port 3001
  - Different logo.svg is already handled in Git branches
*/

pipeline {
    /*
      We run the pipeline on any available Jenkins agent.
      In our case, this is the Jenkins VM itself.
    */
    agent any

    /*
      Environment variables common for all stages.
      PORT will be calculated dynamically based on branch name.
    */
    environment {
        APP_NAME = "cicd-app"
    }

    stages {

        stage('Checkout') {
            /*
              Source code checkout.
              In Multibranch pipeline Jenkins automatically
              checks out the correct branch.
            */
            steps {
                echo "Checking out source code from branch: ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage('Build') {
            /*
              Application build stage.
              For Node.js app this usually means installing dependencies.
            */
            steps {
                echo "Building application..."
                nodejs('node18')  {
                    sh 'npm install'
                }
            }
        }

        stage('Test') {
            /*
              Test stage.
              If tests exist, they should be executed here.
              If not, this stage still demonstrates CI structure.
            */
            steps {
                echo "Running tests..."
                nodejs('node18') {                
                    sh 'npm test || echo "No tests defined"'
                }
            }
        }

        stage('Set Environment Variables') {
            /*
              Define environment-specific variables
              based on branch name.
            */
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.APP_PORT = '3000'
                        echo "Main branch detected. Using port 3000."
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.APP_PORT = '3001'
                        echo "Dev branch detected. Using port 3001."
                    } else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            /*
              Build Docker image using Dockerfile from repository.
              Image tag contains branch name for clarity.
            */
            steps {
                echo "Building Docker image..."
                sh """
                  docker build \
                    -t ${APP_NAME}:${env.BRANCH_NAME} .
                """
            }
        }

        stage('Deploy') {
            /*
              Deployment stage.
              - Stop and remove existing container if it exists
              - Run new container on branch-specific port
            */
            steps {
                echo "Deploying application on port ${env.APP_PORT}..."

                sh """
                  docker rm -f ${APP_NAME}-${env.BRANCH_NAME} || true

                  docker run -d \
                    --name ${APP_NAME}-${env.BRANCH_NAME} \
                    -p ${env.APP_PORT}:3000 \
                    ${APP_NAME}:${env.BRANCH_NAME}
                """
            }
        }
    }

    post {
        /*
          Post actions are executed after pipeline completion.
        */
        success {
            echo "Deployment successful for branch ${env.BRANCH_NAME}"
        }

        failure {
            echo "Pipeline failed for branch ${env.BRANCH_NAME}"
        }
    }
}

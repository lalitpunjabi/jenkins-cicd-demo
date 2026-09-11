pipeline {

    agent any

    parameters {

        booleanParam(
            name: 'SIMULATE_FAILURE',
            defaultValue: false,
            description: 'Simulate deployment failure and trigger automatic rollback'
        )
    }

    environment {

        APP_NAME = 'jenkins-cicd-demo'

        DOCKER_IMAGE = 'lalit0409/jenkins-cicd-demo'

        IMAGE_TAG = "${BUILD_NUMBER}"

        CONTAINER_1 = 'jenkins-app-1'

        CONTAINER_2 = 'jenkins-app-2'

        PORT_1 = '5001'

        PORT_2 = '5002'
    }

    stages {

        /*
         * ==========================
         * CHECKOUT
         * ==========================
         */

        stage('Checkout') {

            steps {

                echo 'Checking out source code...'

                checkout scm
            }
        }


        /*
         * ==========================
         * BUILD
         * ==========================
         */

        stage('Build') {

            steps {

                echo 'Installing application dependencies...'

                sh '''
                    python3 -m venv venv

                    ./venv/bin/pip install --upgrade pip

                    ./venv/bin/pip install -r requirements.txt
                '''
            }
        }


        /*
         * ==========================
         * TEST
         * ==========================
         */

        stage('Test') {

            steps {

                echo 'Running automated tests...'

                sh '''
                    ./venv/bin/pytest
                '''
            }
        }


        /*
         * ==========================
         * PACKAGE
         * ==========================
         */

        stage('Package') {

            steps {

                echo 'Packaging application...'

                sh '''
                    tar -czf ${APP_NAME}-${BUILD_NUMBER}.tar.gz \
                    app.py requirements.txt Dockerfile
                '''
            }
        }


        /*
         * ==========================
         * DOCKER BUILD
         * ==========================
         */

        stage('Docker Build') {

            steps {

                echo "Building Docker image: ${DOCKER_IMAGE}:${IMAGE_TAG}"

                sh '''
                    docker build \
                    -t ${DOCKER_IMAGE}:${IMAGE_TAG} \
                    -t ${DOCKER_IMAGE}:latest \
                    .
                '''
            }
        }


        /*
         * ==========================
         * DOCKER PUSH
         * ==========================
         */

        stage('Docker Login & Push') {

            steps {

                withCredentials([

                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',

                        usernameVariable: 'DOCKER_USERNAME',

                        passwordVariable: 'DOCKER_PASSWORD'
                    )

                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USERNAME" \
                        --password-stdin

                        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}

                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }


        /*
         * ==========================
         * SAVE PREVIOUS VERSION
         * ==========================
         */

        stage('Save Previous Version') {

            steps {

                echo 'Saving currently deployed version for rollback...'

                sh '''
                    if docker image inspect ${DOCKER_IMAGE}:current >/dev/null 2>&1
                    then

                        docker tag \
                        ${DOCKER_IMAGE}:current \
                        ${DOCKER_IMAGE}:previous

                        echo "Previous version saved."

                    else

                        echo "No current version found."
                        echo "This may be the first deployment."

                    fi
                '''
            }
        }


        /*
         * ==========================
         * PULL NEW IMAGE
         * ==========================
         */

        stage('Pull New Image') {

            steps {

                echo "Pulling ${DOCKER_IMAGE}:${IMAGE_TAG}"

                sh '''
                    docker pull ${DOCKER_IMAGE}:${IMAGE_TAG}
                '''
            }
        }


        /*
         * ==========================
         * ROLLING DEPLOYMENT APP 1
         * ==========================
         */

        stage('Rolling Deployment - App 1') {

            steps {

                echo "Deploying version ${IMAGE_TAG} to App 1..."

                sh '''
                    docker stop ${CONTAINER_1} || true

                    docker rm ${CONTAINER_1} || true

                    docker run -d \
                        --name ${CONTAINER_1} \
                        -p ${PORT_1}:5000 \
                        ${DOCKER_IMAGE}:${IMAGE_TAG}
                '''

                echo 'Waiting for App 1 to start...'

                sleep 10

                echo 'Checking App 1 health...'

                sh '''
                    curl -f http://localhost:${PORT_1}/health
                '''

                echo 'App 1 deployment successful.'
            }
        }


        /*
         * ==========================
         * CONTROLLED FAILURE
         * ==========================
         */

        stage('Controlled Failure') {

            when {

                expression {

                    return params.SIMULATE_FAILURE
                }
            }

            steps {

                echo '======================================='

                echo 'CONTROLLED FAILURE ENABLED'

                echo '======================================='

                sh '''
                    echo "Simulating deployment failure..."

                    exit 1
                '''
            }
        }


        /*
         * ==========================
         * ROLLING DEPLOYMENT APP 2
         * ==========================
         */

        stage('Rolling Deployment - App 2') {

            steps {

                echo "Deploying version ${IMAGE_TAG} to App 2..."

                sh '''
                    docker stop ${CONTAINER_2} || true

                    docker rm ${CONTAINER_2} || true

                    docker run -d \
                        --name ${CONTAINER_2} \
                        -p ${PORT_2}:5000 \
                        ${DOCKER_IMAGE}:${IMAGE_TAG}
                '''

                echo 'Waiting for App 2 to start...'

                sleep 10

                echo 'Checking App 2 health...'

                sh '''
                    curl -f http://localhost:${PORT_2}/health
                '''

                echo 'App 2 deployment successful.'
            }
        }


        /*
         * ==========================
         * UPDATE CURRENT VERSION
         * ==========================
         */

        stage('Update Current Version') {

            steps {

                echo "Marking ${IMAGE_TAG} as current production version..."

                sh '''
                    docker tag \
                    ${DOCKER_IMAGE}:${IMAGE_TAG} \
                    ${DOCKER_IMAGE}:current
                '''
            }
        }


        /*
         * ==========================
         * FINAL VERIFICATION
         * ==========================
         */

        stage('Final Verification') {

            steps {

                echo 'Performing final deployment verification...'

                sh '''
                    curl -f http://localhost:5001/health

                    curl -f http://localhost:5002/health

                    curl -f http://localhost/health
                '''

                echo 'Final verification successful.'
            }
        }
    }


    /*
     * ==========================
     * POST ACTIONS
     * ==========================
     */

    post {

        success {

            echo '''
========================================
      CI/CD PIPELINE SUCCESSFUL
========================================
'''
            echo "Application: ${APP_NAME}"

            echo "Deployed Version: ${IMAGE_TAG}"

            echo "Docker Image: ${DOCKER_IMAGE}:${IMAGE_TAG}"
        }


        failure {

            echo '''
========================================
       DEPLOYMENT FAILED
       STARTING ROLLBACK
========================================
'''

            sh '''
                if docker image inspect ${DOCKER_IMAGE}:previous >/dev/null 2>&1
                then

                    echo "Previous working image found."

                    echo "Starting rollback..."

                    docker stop ${CONTAINER_1} || true

                    docker rm ${CONTAINER_1} || true

                    docker stop ${CONTAINER_2} || true

                    docker rm ${CONTAINER_2} || true


                    echo "Restoring App 1..."

                    docker run -d \
                        --name ${CONTAINER_1} \
                        -p ${PORT_1}:5000 \
                        ${DOCKER_IMAGE}:previous


                    echo "Restoring App 2..."

                    docker run -d \
                        --name ${CONTAINER_2} \
                        -p ${PORT_2}:5000 \
                        ${DOCKER_IMAGE}:previous


                    echo "Waiting for rollback containers..."

                    sleep 10


                    echo "Checking rollback health..."

                    curl -f http://localhost:${PORT_1}/health

                    curl -f http://localhost:${PORT_2}/health


                    echo "Restoring current tag..."

                    docker tag \
                        ${DOCKER_IMAGE}:previous \
                        ${DOCKER_IMAGE}:current


                    echo '''
========================================
        ROLLBACK SUCCESSFUL
========================================
'''

                else

                    echo '''
========================================
       NO PREVIOUS VERSION
       ROLLBACK NOT POSSIBLE
========================================

This may be the first deployment.
'''

                fi
            '''
        }


        always {

            echo "Build Number: ${BUILD_NUMBER}"

            echo "Application: ${APP_NAME}"

            echo "Docker Image: ${DOCKER_IMAGE}:${IMAGE_TAG}"
        }
    }
}
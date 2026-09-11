pipeline {

    agent any

    parameters {

        booleanParam(
            name: 'SIMULATE_FAILURE',
            defaultValue: false,
            description: 'Simulate a deployment failure to test automatic rollback'
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
         * ============================
         * CHECKOUT
         * ============================
         */

        stage('Checkout') {

            steps {

                echo 'Checking out source code from GitHub...'

                checkout scm
            }
        }


        /*
         * ============================
         * BUILD
         * ============================
         */

        stage('Build') {

            steps {

                echo 'Installing Python dependencies...'

                sh '''
                    python3 -m venv venv

                    ./venv/bin/pip install --upgrade pip

                    ./venv/bin/pip install -r requirements.txt
                '''
            }
        }


        /*
         * ============================
         * TEST
         * ============================
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
         * ============================
         * PACKAGE
         * ============================
         */

        stage('Package') {

            steps {

                echo 'Packaging application...'

                sh '''
                    tar -czf ${APP_NAME}-${BUILD_NUMBER}.tar.gz \
                        app.py \
                        requirements.txt \
                        Dockerfile
                '''
            }
        }


        /*
         * ============================
         * DOCKER BUILD
         * ============================
         */

        stage('Docker Build') {

            steps {

                echo "Building Docker image ${DOCKER_IMAGE}:${IMAGE_TAG}..."

                sh '''
                    docker build \
                        -t ${DOCKER_IMAGE}:${IMAGE_TAG} \
                        -t ${DOCKER_IMAGE}:latest \
                        .
                '''
            }
        }


        /*
         * ============================
         * DOCKER HUB PUSH
         * ============================
         */

        stage('Docker Login & Push') {

            steps {

                echo 'Logging in to Docker Hub and pushing image...'

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
         * ============================
         * SAVE CURRENT VERSION
         * ============================
         *
         * Automatically detects the image
         * currently used by the running
         * application and saves it as
         * "previous" for rollback.
         */

        stage('Save Previous Version') {

            steps {

                echo 'Detecting currently deployed version...'

                sh '''
                    if docker inspect ${CONTAINER_1} >/dev/null 2>&1
                    then

                        CURRENT_IMAGE=$(docker inspect \
                            ${CONTAINER_1} \
                            --format='{{.Config.Image}}')

                        echo "Currently deployed image: $CURRENT_IMAGE"

                        docker tag \
                            "$CURRENT_IMAGE" \
                            ${DOCKER_IMAGE}:previous

                        echo "Previous version saved successfully."

                    else

                        echo "No existing deployment found."

                        echo "This appears to be the first deployment."

                    fi
                '''
            }
        }


        /*
         * ============================
         * PULL NEW IMAGE
         * ============================
         */

        stage('Pull New Image') {

            steps {

                echo "Preparing image ${DOCKER_IMAGE}:${IMAGE_TAG}..."

                sh '''
                    docker pull ${DOCKER_IMAGE}:${IMAGE_TAG}
                '''
            }
        }


        /*
         * ============================
         * ROLLING DEPLOYMENT - APP 1
         * ============================
         */

        stage('Rolling Deployment - App 1') {

            steps {

                echo "Deploying ${IMAGE_TAG} to App 1..."

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
                    for i in $(seq 1 15)
                    do
                        if curl -fsS http://localhost:${PORT_1}/health
                        then
                            echo "App 1 is healthy."
                            exit 0
                        fi

                        echo "Waiting for App 1..."

                        sleep 2
                    done

                    echo "App 1 health check failed."

                    exit 1
                '''

                echo 'App 1 deployment completed successfully.'
            }
        }


        /*
         * ============================
         * CONTROLLED FAILURE
         * ============================
         *
         * This stage is optional.
         *
         * When SIMULATE_FAILURE = true,
         * the pipeline intentionally fails
         * here and Jenkins automatically
         * executes the rollback section.
         */

        stage('Controlled Failure') {

            when {

                expression {

                    return params.SIMULATE_FAILURE
                }
            }

            steps {

                echo '========================================'

                echo 'CONTROLLED FAILURE ENABLED'

                echo 'Deployment failure will be simulated.'

                echo 'Automatic rollback will start.'

                echo '========================================'

                sh '''
                    echo "Simulating deployment failure..."

                    exit 1
                '''
            }
        }


        /*
         * ============================
         * ROLLING DEPLOYMENT - APP 2
         * ============================
         */

        stage('Rolling Deployment - App 2') {

            steps {

                echo "Deploying ${IMAGE_TAG} to App 2..."

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
                    for i in $(seq 1 15)
                    do
                        if curl -fsS http://localhost:${PORT_2}/health
                        then
                            echo "App 2 is healthy."
                            exit 0
                        fi

                        echo "Waiting for App 2..."

                        sleep 2
                    done

                    echo "App 2 health check failed."

                    exit 1
                '''

                echo 'App 2 deployment completed successfully.'
            }
        }


        /*
         * ============================
         * UPDATE CURRENT VERSION
         * ============================
         */

        stage('Update Current Version') {

            steps {

                echo "Marking ${IMAGE_TAG} as the current version..."

                sh '''
                    docker tag \
                        ${DOCKER_IMAGE}:${IMAGE_TAG} \
                        ${DOCKER_IMAGE}:current

                    echo "Current version updated successfully."
                '''
            }
        }


        /*
         * ============================
         * FINAL VERIFICATION
         * ============================
         */

        stage('Final Verification') {

            steps {

                echo 'Running final deployment verification...'

                sh '''
                    echo "Checking App 1..."

                    curl -fsS http://localhost:${PORT_1}/health

                    echo ""

                    echo "Checking App 2..."

                    curl -fsS http://localhost:${PORT_2}/health

                    echo ""

                    echo "Checking application through Nginx..."

                    curl -fsS http://localhost/health

                    echo ""

                    echo "All health checks passed."
                '''
            }
        }
    }


    /*
     * ================================
     * POST ACTIONS
     * ================================
     */

    post {

        /*
         * ============================
         * SUCCESS
         * ============================
         */

        success {

            echo '========================================'

            echo '       CI/CD PIPELINE SUCCESSFUL'

            echo '========================================'

            echo "Application: ${APP_NAME}"

            echo "Deployed Version: ${IMAGE_TAG}"

            echo "Docker Image: ${DOCKER_IMAGE}:${IMAGE_TAG}"

            echo 'Rolling deployment completed successfully.'
        }


        /*
         * ============================
         * FAILURE + AUTOMATIC ROLLBACK
         * ============================
         */

        failure {

            echo '========================================'

            echo '       DEPLOYMENT FAILED'

            echo '       STARTING AUTOMATIC ROLLBACK'

            echo '========================================'

            sh '''
                if docker image inspect ${DOCKER_IMAGE}:previous >/dev/null 2>&1
                then

                    echo "Previous working image found."

                    PREVIOUS_IMAGE=$(docker image inspect \
                        ${DOCKER_IMAGE}:previous \
                        --format='{{.Id}}')

                    echo "Previous image ID: $PREVIOUS_IMAGE"

                    echo "Stopping current App 1..."

                    docker stop ${CONTAINER_1} || true

                    docker rm ${CONTAINER_1} || true


                    echo "Stopping current App 2..."

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


                    echo "Checking App 1 after rollback..."

                    for i in $(seq 1 15)
                    do
                        if curl -fsS http://localhost:${PORT_1}/health
                        then
                            echo "App 1 rollback successful."
                            break
                        fi

                        echo "Waiting for App 1 rollback..."

                        sleep 2

                    done


                    echo "Checking App 2 after rollback..."

                    for i in $(seq 1 15)
                    do
                        if curl -fsS http://localhost:${PORT_2}/health
                        then
                            echo "App 2 rollback successful."
                            break
                        fi

                        echo "Waiting for App 2 rollback..."

                        sleep 2

                    done


                    echo "Restoring current tag..."

                    docker tag \
                        ${DOCKER_IMAGE}:previous \
                        ${DOCKER_IMAGE}:current


                    echo "========================================"

                    echo "        ROLLBACK COMPLETED"

                    echo "========================================"

                else

                    echo "========================================"

                    echo "       NO PREVIOUS VERSION"

                    echo "       ROLLBACK NOT POSSIBLE"

                    echo "========================================"

                    echo "No previous deployment was found."

                fi
            '''
        }


        /*
         * ============================
         * ALWAYS
         * ============================
         */

        always {

            echo '========================================'

            echo "Build Number: ${BUILD_NUMBER}"

            echo "Application: ${APP_NAME}"

            echo "Docker Image: ${DOCKER_IMAGE}:${IMAGE_TAG}"

            echo '========================================'
        }
    }
}
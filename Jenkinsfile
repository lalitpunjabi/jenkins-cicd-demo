pipeline {

    agent any

    parameters {

        booleanParam(
            name: 'SIMULATE_FAILURE',
            defaultValue: false,
            description: 'Simulate a deployment failure to test automatic rollback'
        )

        booleanParam(
            name: 'RUN_SECURITY_SCANS',
            defaultValue: true,
            description: 'Run DevSecOps security scans'
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

        SECURITY_DIR = 'security'

        K8S_DIR = 'k8s'
    }

    stages {

        /*
         * ============================================================
         * CHECKOUT
         * ============================================================
         */

        stage('Checkout') {

            steps {

                echo 'Checking out source code from GitHub...'

                checkout scm
            }
        }


        /*
         * ============================================================
         * ENVIRONMENT VALIDATION
         * ============================================================
         */

        stage('Environment Validation') {

            steps {

                sh '''
                    set -e

                    echo "========================================"
                    echo "      ENVIRONMENT VALIDATION"
                    echo "========================================"

                    echo "Python:"
                    python3 --version

                    echo "Docker:"
                    docker --version

                    echo "Git:"
                    git --version

                    echo "Kubectl:"
                    kubectl version --client --output=yaml 2>/dev/null || true

                    echo "Trivy:"
                    trivy --version

                    echo "Gitleaks:"
                    gitleaks version

                    echo "Yamllint:"
                    yamllint --version

                    echo "========================================"
                '''
            }
        }


        /*
         * ============================================================
         * PREPARE SECURITY DIRECTORIES
         * ============================================================
         */

        stage('Prepare Security Workspace') {

            when {

                expression {

                    return params.RUN_SECURITY_SCANS
                }
            }

            steps {

                sh '''
                    set -e

                    mkdir -p ${SECURITY_DIR}

                    rm -f ${SECURITY_DIR}/*.txt
                    rm -f ${SECURITY_DIR}/*.json
                    rm -f ${SECURITY_DIR}/*.sarif

                    echo "Security workspace prepared."
                '''
            }
        }


        /*
         * ============================================================
         * YAML VALIDATION
         * ============================================================
         */

        stage('YAML Validation') {

            when {

                expression {

                    return params.RUN_SECURITY_SCANS
                }
            }

            steps {

                echo 'Validating YAML configuration...'

                sh '''
                    set -e

                    if [ -d "${K8S_DIR}" ]
                    then

                        echo "Running yamllint..."

                        yamllint ${K8S_DIR}/ \
                            | tee ${SECURITY_DIR}/yamllint.txt

                        echo "Running Kubernetes dry-run..."

                        kubectl apply \
                            --dry-run=client \
                            -f ${K8S_DIR}/ \
                            | tee ${SECURITY_DIR}/kubectl-dry-run.txt

                    else

                        echo "WARNING: k8s directory not found."

                    fi
                '''
            }
        }


        /*
         * ============================================================
         * BUILD
         * ============================================================
         */

        stage('Build') {

            steps {

                echo 'Creating Python virtual environment...'

                sh '''
                    set -e

                    rm -rf venv

                    python3 -m venv venv

                    ./venv/bin/pip install --upgrade pip

                    ./venv/bin/pip install -r requirements.txt

                    echo "Python dependencies installed successfully."
                '''
            }
        }


        /*
         * ============================================================
         * UNIT TESTS
         * ============================================================
         */

        stage('Test') {

            steps {

                echo 'Running automated tests...'

                sh '''
                    set -e

                    ./venv/bin/pytest \
                        -v \
                        --junitxml=test-results.xml
                '''
            }

            post {

                always {

                    junit(
                        allowEmptyResults: true,
                        testResults: 'test-results.xml'
                    )
                }
            }
        }


        /*
         * ============================================================
         * DEPENDENCY SECURITY
         * ============================================================
         */

        stage('Dependency Security Scan') {

            when {

                expression {

                    return params.RUN_SECURITY_SCANS
                }
            }

            steps {

                echo 'Running pip-audit...'

                sh '''
                    set -e

                    ./venv/bin/pip install pip-audit

                    ./venv/bin/pip-audit \
                        -r requirements.txt \
                        | tee ${SECURITY_DIR}/pip-audit.txt
                '''
            }
        }


        /*
         * ============================================================
         * SECRET SCANNING
         * ============================================================
         */

        stage('Secret Scan') {

            when {

                expression {

                    return params.RUN_SECURITY_SCANS
                }
            }

            steps {

                echo 'Scanning repository for secrets...'

                sh '''
                    set -e

                    gitleaks detect \
                        --no-banner \
                        --redact \
                        --report-format json \
                        --report-path ${SECURITY_DIR}/gitleaks.json
                '''
            }
        }


        /*
         * ============================================================
         * PACKAGE
         * ============================================================
         */

        stage('Package') {

            steps {

                echo 'Packaging application...'

                sh '''
                    set -e

                    tar -czf ${APP_NAME}-${BUILD_NUMBER}.tar.gz \
                        app.py \
                        requirements.txt \
                        Dockerfile
                '''
            }
        }


        /*
         * ============================================================
         * DOCKERFILE SECURITY SCAN
         * ============================================================
         */

        stage('Dockerfile Security Scan') {

            when {

                expression {

                    return params.RUN_SECURITY_SCANS
                }
            }

            steps {

                echo 'Scanning Dockerfile with Trivy...'

                sh '''
                    set -e

                    trivy config \
                        --severity HIGH,CRITICAL \
                        --exit-code 1 \
                        Dockerfile \
                        | tee ${SECURITY_DIR}/trivy-dockerfile.txt
                '''
            }
        }


        /*
         * ============================================================
         * KUBERNETES SECURITY SCAN
         * ============================================================
         */

        stage('Kubernetes Security Scan') {

            when {

                expression {

                    return params.RUN_SECURITY_SCANS
                }
            }

            steps {

                sh '''
                    set -e

                    if [ -d "${K8S_DIR}" ]
                    then

                        trivy config \
                            --severity HIGH,CRITICAL \
                            --exit-code 1 \
                            ${K8S_DIR}/ \
                            | tee ${SECURITY_DIR}/trivy-k8s.txt

                    else

                        echo "WARNING: No Kubernetes directory found."

                    fi
                '''
            }
        }


        /*
         * ============================================================
         * DOCKER BUILD
         * ============================================================
         */

        stage('Docker Build') {

            steps {

                echo "Building Docker image ${DOCKER_IMAGE}:${IMAGE_TAG}..."

                sh '''
                    set -e

                    docker build \
                        --pull \
                        --no-cache \
                        -t ${DOCKER_IMAGE}:${IMAGE_TAG} \
                        .
                '''
            }
        }


        /*
         * ============================================================
         * DOCKER IMAGE SECURITY SCAN
         * ============================================================
         */

        stage('Container Security Scan') {

            when {

                expression {

                    return params.RUN_SECURITY_SCANS
                }
            }

            steps {

                echo 'Scanning Docker image for vulnerabilities...'

                sh '''
                    set -e

                    trivy image \
                        --severity HIGH,CRITICAL \
                        --exit-code 1 \
                        --ignore-unfixed \
                        ${DOCKER_IMAGE}:${IMAGE_TAG} \
                        | tee ${SECURITY_DIR}/trivy-image.txt
                '''
            }
        }


        /*
         * ============================================================
         * FILESYSTEM SECURITY SCAN
         * ============================================================
         */

        stage('Filesystem Security Scan') {

            when {

                expression {

                    return params.RUN_SECURITY_SCANS
                }
            }

            steps {

                echo 'Scanning source filesystem with Trivy...'

                sh '''
                    set -e

                    trivy fs \
                        --severity HIGH,CRITICAL \
                        --exit-code 1 \
                        --ignore-unfixed \
                        . \
                        | tee ${SECURITY_DIR}/trivy-filesystem.txt
                '''
            }
        }


        /*
         * ============================================================
         * TAG LATEST
         * ============================================================
         */

        stage('Tag Docker Image') {

            steps {

                sh '''
                    set -e

                    docker tag \
                        ${DOCKER_IMAGE}:${IMAGE_TAG} \
                        ${DOCKER_IMAGE}:latest

                    echo "Docker image tagged successfully."
                '''
            }
        }


        /*
         * ============================================================
         * SAVE PREVIOUS VERSION
         * ============================================================
         */

        stage('Save Previous Version') {

            steps {

                echo 'Detecting currently deployed version...'

                sh '''
                    set -e

                    if docker inspect ${CONTAINER_1} >/dev/null 2>&1
                    then

                        CURRENT_IMAGE=$(docker inspect \
                            ${CONTAINER_1} \
                            --format='{{.Config.Image}}')

                        echo "Currently deployed image:"
                        echo "$CURRENT_IMAGE"

                        if docker image inspect "$CURRENT_IMAGE" >/dev/null 2>&1
                        then

                            docker tag \
                                "$CURRENT_IMAGE" \
                                ${DOCKER_IMAGE}:previous

                            echo "Previous version saved."

                        else

                            echo "WARNING: Current image no longer exists locally."

                        fi

                    else

                        echo "No existing deployment found."

                        echo "This is the first deployment."

                    fi
                '''
            }
        }


        /*
         * ============================================================
         * DOCKER HUB LOGIN AND PUSH
         * ============================================================
         */

        stage('Docker Login & Push') {

            steps {

                echo 'Logging in to Docker Hub...'

                withCredentials([

                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )

                ]) {

                    sh '''

                        set +x

                        echo "$DOCKER_PASSWORD" | docker login \
                            --username "$DOCKER_USERNAME" \
                            --password-stdin

                        set -e

                        docker push \
                            ${DOCKER_IMAGE}:${IMAGE_TAG}

                        docker push \
                            ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }


        /*
         * ============================================================
         * PULL IMAGE
         * ============================================================
         */

        stage('Pull New Image') {

            steps {

                sh '''
                    set -e

                    docker pull \
                        ${DOCKER_IMAGE}:${IMAGE_TAG}
                '''
            }
        }


        /*
         * ============================================================
         * DEPLOY APP 1
         * ============================================================
         */

        stage('Rolling Deployment - App 1') {

            steps {

                echo "Deploying version ${IMAGE_TAG} to App 1..."

                sh '''
                    set -e

                    docker stop ${CONTAINER_1} || true

                    docker rm ${CONTAINER_1} || true

                    docker run -d \
                        --name ${CONTAINER_1} \
                        --restart unless-stopped \
                        -p ${PORT_1}:5000 \
                        ${DOCKER_IMAGE}:${IMAGE_TAG}
                '''

                sleep 5

                sh '''
                    set -e

                    for i in $(seq 1 15)
                    do

                        if curl -fsS \
                            http://localhost:${PORT_1}/health
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
            }
        }


        /*
         * ============================================================
         * CONTROLLED FAILURE
         * ============================================================
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
                echo 'Automatic rollback will be triggered.'
                echo '========================================'

                sh '''
                    echo "Simulating deployment failure..."

                    exit 1
                '''
            }
        }


        /*
         * ============================================================
         * DEPLOY APP 2
         * ============================================================
         */

        stage('Rolling Deployment - App 2') {

            steps {

                echo "Deploying version ${IMAGE_TAG} to App 2..."

                sh '''
                    set -e

                    docker stop ${CONTAINER_2} || true

                    docker rm ${CONTAINER_2} || true

                    docker run -d \
                        --name ${CONTAINER_2} \
                        --restart unless-stopped \
                        -p ${PORT_2}:5000 \
                        ${DOCKER_IMAGE}:${IMAGE_TAG}
                '''

                sleep 5

                sh '''
                    set -e

                    for i in $(seq 1 15)
                    do

                        if curl -fsS \
                            http://localhost:${PORT_2}/health
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
            }
        }


        /*
         * ============================================================
         * UPDATE CURRENT VERSION
         * ============================================================
         */

        stage('Update Current Version') {

            steps {

                sh '''
                    set -e

                    docker tag \
                        ${DOCKER_IMAGE}:${IMAGE_TAG} \
                        ${DOCKER_IMAGE}:current

                    echo "Current version updated."
                '''
            }
        }


        /*
         * ============================================================
         * FINAL VERIFICATION
         * ============================================================
         */

        stage('Final Verification') {

            steps {

                echo 'Running final deployment verification...'

                sh '''
                    set -e

                    echo "Checking App 1..."

                    curl -fsS \
                        http://localhost:${PORT_1}/health

                    echo ""

                    echo "Checking App 2..."

                    curl -fsS \
                        http://localhost:${PORT_2}/health

                    echo ""

                    if curl -fsS http://localhost/health
                    then

                        echo ""
                        echo "Nginx health check passed."

                    else

                        echo ""
                        echo "Nginx health check unavailable."

                        echo "Direct application checks passed."

                    fi

                    echo ""
                    echo "All required health checks passed."
                '''
            }
        }


        /*
         * ============================================================
         * DOCKER STATUS
         * ============================================================
         */

        stage('Deployment Status') {

            steps {

                sh '''
                    echo "========================================"
                    echo "       DEPLOYMENT STATUS"
                    echo "========================================"

                    docker ps \
                        --filter "name=${CONTAINER_1}" \
                        --filter "name=${CONTAINER_2}"

                    echo "========================================"
                '''
            }
        }
    }


    /*
     * ================================================================
     * POST ACTIONS
     * ================================================================
     */

    post {


        /*
         * ============================================================
         * SUCCESS
         * ============================================================
         */

        success {

            echo '========================================'
            echo '       CI/CD PIPELINE SUCCESSFUL'
            echo '========================================'

            echo "Application: ${APP_NAME}"

            echo "Build Number: ${BUILD_NUMBER}"

            echo "Docker Image: ${DOCKER_IMAGE}:${IMAGE_TAG}"

            echo 'Security validation completed.'

            echo 'Deployment completed successfully.'

            archiveArtifacts(
                artifacts: "${APP_NAME}-${BUILD_NUMBER}.tar.gz,security/*",
                allowEmptyArchive: true,
                fingerprint: true
            )
        }


        /*
         * ============================================================
         * FAILURE + AUTOMATIC ROLLBACK
         * ============================================================
         */

        failure {

            echo '========================================'
            echo '       PIPELINE FAILED'
            echo '========================================'

            sh '''
                set +e

                echo "Starting automatic rollback..."

                if docker image inspect \
                    ${DOCKER_IMAGE}:previous >/dev/null 2>&1
                then

                    echo "Previous working image found."

                    echo "Stopping current App 1..."

                    docker stop ${CONTAINER_1} || true

                    docker rm ${CONTAINER_1} || true


                    echo "Stopping current App 2..."

                    docker stop ${CONTAINER_2} || true

                    docker rm ${CONTAINER_2} || true


                    echo "Restoring App 1..."

                    docker run -d \
                        --name ${CONTAINER_1} \
                        --restart unless-stopped \
                        -p ${PORT_1}:5000 \
                        ${DOCKER_IMAGE}:previous


                    echo "Restoring App 2..."

                    docker run -d \
                        --name ${CONTAINER_2} \
                        --restart unless-stopped \
                        -p ${PORT_2}:5000 \
                        ${DOCKER_IMAGE}:previous


                    echo "Waiting for rollback containers..."

                    sleep 5


                    echo "Checking App 1 rollback..."

                    APP1_OK=0

                    for i in $(seq 1 15)
                    do

                        if curl -fsS \
                            http://localhost:${PORT_1}/health
                        then

                            echo "App 1 rollback successful."

                            APP1_OK=1

                            break

                        fi

                        sleep 2

                    done


                    echo "Checking App 2 rollback..."

                    APP2_OK=0

                    for i in $(seq 1 15)
                    do

                        if curl -fsS \
                            http://localhost:${PORT_2}/health
                        then

                            echo "App 2 rollback successful."

                            APP2_OK=1

                            break

                        fi

                        sleep 2

                    done


                    if [ "$APP1_OK" -eq 1 ] && [ "$APP2_OK" -eq 1 ]
                    then

                        docker tag \
                            ${DOCKER_IMAGE}:previous \
                            ${DOCKER_IMAGE}:current

                        echo "========================================"
                        echo "        ROLLBACK COMPLETED"
                        echo "========================================"

                    else

                        echo "========================================"
                        echo "        ROLLBACK FAILED"
                        echo "========================================"

                    fi

                else

                    echo "========================================"
                    echo "       NO PREVIOUS VERSION"
                    echo "========================================"

                    echo "Rollback is not possible because this"
                    echo "is the first deployment."

                fi
            '''

            archiveArtifacts(
                artifacts: 'security/*',
                allowEmptyArchive: true,
                fingerprint: true
            )
        }


        /*
         * ============================================================
         * ALWAYS
         * ============================================================
         */

        always {

            echo '========================================'
            echo "Build Number: ${BUILD_NUMBER}"
            echo "Application: ${APP_NAME}"
            echo "Docker Image: ${DOCKER_IMAGE}:${IMAGE_TAG}"
            echo '========================================'

            sh '''
                echo "Cleaning temporary Python cache..."

                rm -rf .pytest_cache
                rm -rf __pycache__

                echo "Workspace cleanup completed."
            '''
        }
    }
}
pipeline {
    environment {
        DOCKER_ID = "simplice"
        DOCKER_TAG = "v.${BUILD_ID}.0"
        MOVIE_IMAGE = "${DOCKER_ID}/movie-service:${DOCKER_TAG}"
        CAST_IMAGE  = "${DOCKER_ID}/cast-service:${DOCKER_TAG}"
    }

    agent any

    stages {
        stage('Docker Build') {
            steps {
                script {
                    sh """
                        docker build -t ${MOVIE_IMAGE} ./movie-service
                        docker build -t ${CAST_IMAGE} ./cast-service
                    """
                }
            }
        }

        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh """
                        docker login -u ${DOCKER_ID} -p ${DOCKER_PASS}
                        docker push ${MOVIE_IMAGE}
                        docker push ${CAST_IMAGE}
                    """
                }
            }
        }

        stage('Deploy to dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh """
                        helm upgrade --install app-dev ./charts \
			  --kubeconfig /var/lib/jenkins/.kube/config \
                          --set movieService.image.tag=${DOCKER_TAG} \
                          --set castService.image.tag=${DOCKER_TAG} \
                          --namespace dev --create-namespace
                    """
                }
            }
        }

        stage('Deploy to QA') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh """
                        mkdir -p .kube
                        cat ${KUBECONFIG} > .kube/config
                        helm upgrade --install app-qa ./charts \
                          --set movieService.image.tag=${DOCKER_TAG} \
                          --set castService.image.tag=${DOCKER_TAG} \
                          --namespace qa --create-namespace
                    """
                }
            }
        }

        stage('Deploy to staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh """
                        mkdir -p .kube
                        cat ${KUBECONFIG} > .kube/config
                        helm upgrade --install app-staging ./charts \
                          --set movieService.image.tag=${DOCKER_TAG} \
                          --set castService.image.tag=${DOCKER_TAG} \
                          --namespace staging --create-namespace
                    """
                }
            }
        }

        stage('Deploy to prod') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Voulez-vous déployer en production ?', ok: 'Yes'
                }
                script {
                    sh """
                        mkdir -p .kube
                        cat ${KUBECONFIG} > .kube/config
                        helm upgrade --install app-prod ./charts \
                          --set movieService.image.tag=${DOCKER_TAG} \
                          --set castService.image.tag=${DOCKER_TAG} \
                          --namespace prod --create-namespace
                    """
                }
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed"
            mail to: "oumclagbaix@gmail.com",
                subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
                body: "Check console output at ${env.BUILD_URL}"
        }

    }
}

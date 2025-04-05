pipeline {
    agent any

    environment {
        BACKEND_IMAGE_NAME  = "hotel-room-booking-and-management-backend"
        FRONTEND_IMAGE_NAME = "hotel-room-booking-and-management-frontend"
        IMAGE_TAG           = "latest"
        DOCKER_REPO         = "shandeep04"
        KUBECONFIG          = "/var/lib/jenkins/.kube/config"
        MINIKUBE_DIR        = "/home/shandeep/.minikube"
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/Shandeepsugumar/Devops_miniproject.git'
                }
            }
        }

        stage('Build Backend Image') {
            steps {
                script {
                    dockerImageBackend = docker.build("${DOCKER_REPO}/${BACKEND_IMAGE_NAME}:${IMAGE_TAG}", "./ReactJS_with_backend_and_frontend-main/backend")
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                script {
                    dockerImageFrontend = docker.build("${DOCKER_REPO}/${FRONTEND_IMAGE_NAME}:${IMAGE_TAG}", "./ReactJS_with_backend_and_frontend-main/frontend")
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credential', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
                        dockerImageBackend.push("${IMAGE_TAG}")
                        dockerImageFrontend.push("${IMAGE_TAG}")
                    }
                }
            }
        }

        stage('Fix Minikube Permissions') {
            steps {
                script {
                    // Temporarily allow Jenkins access to Minikube certs
                    sh """
                        sudo chmod -R a+r ${MINIKUBE_DIR}/ca.crt
                        sudo chmod -R a+r ${MINIKUBE_DIR}/profiles/minikube/client.crt
                        sudo chmod -R a+r ${MINIKUBE_DIR}/profiles/minikube/client.key
                    """
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    withEnv(["KUBECONFIG=${KUBECONFIG}"]) {
                        sh 'kubectl apply -f deploy.yaml --validate=false'
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    withEnv(["KUBECONFIG=${KUBECONFIG}"]) {
                        sh 'kubectl get pods'
                        sh 'kubectl get svc'
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed."
        }
        failure {
            echo "Pipeline failed. Check logs above for details."
        }
        success {
            echo "Pipeline ran successfully. Deployment complete!"
        }
    }
}

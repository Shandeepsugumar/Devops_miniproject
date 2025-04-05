pipeline {
    agent any

    environment {
        BACKEND_IMAGE_NAME  = "hotel-room-booking-and-management-backend"
        FRONTEND_IMAGE_NAME = "hotel-room-booking-and-management-frontend"
        IMAGE_TAG           = "latest"
        DOCKER_REPO         = "shandeep04"
        KUBECONFIG          = "/var/lib/jenkins/.kube/config"
        SHANDEEP_HOME       = "/home/shandeep"
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

        stage('Setup Kubeconfig') {
            steps {
                script {
                    // Copy kube config and certs to Jenkins-accessible location
                    sh """
                        sudo mkdir -p /var/lib/jenkins/.kube
                        sudo mkdir -p /var/lib/jenkins/.minikube/profiles/minikube

                        sudo cp ${SHANDEEP_HOME}/.kube/config /var/lib/jenkins/.kube/
                        sudo cp ${SHANDEEP_HOME}/.minikube/ca.crt /var/lib/jenkins/.minikube/
                        sudo cp ${SHANDEEP_HOME}/.minikube/profiles/minikube/client.crt /var/lib/jenkins/.minikube/profiles/minikube/
                        sudo cp ${SHANDEEP_HOME}/.minikube/profiles/minikube/client.key /var/lib/jenkins/.minikube/profiles/minikube/

                        sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube /var/lib/jenkins/.minikube
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

pipeline {     
    agent any      
    
    environment {         
        BACKEND_IMAGE_NAME  = "hotel-room-booking-and-management-backend"         
        FRONTEND_IMAGE_NAME = "hotel-room-booking-and-management-frontend"         
        IMAGE_TAG           = "latest"         
        DOCKER_REGISTRY     = "docker.io"         
        DOCKER_REPO         = "shandeep04" 
        // Use Jenkins-accessible kubeconfig path
        KUBECONFIG          = "/var/lib/jenkins/.kube/config"    
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
                script {              
                    // Secure login: Avoid exposing password directly (consider using credentials)
                    sh 'docker login -u shandeep04 -p "Shandeep-4621"'             
                    dockerImageBackend.push("${IMAGE_TAG}")              
                    dockerImageFrontend.push("${IMAGE_TAG}")          
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
    }

    post {         
        always {             
            echo "Pipeline execution completed."         
        }     
    } 
}

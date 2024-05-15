pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS = '3e414183-fdfc-4540-840d-c351e2a184b7' 
        FRONTEND_IMAGE_NAME = 'vladhl/frontend'
        BACKEND_IMAGE_NAME = 'vladhl/backend'
        DB_IMAGE_NAME = 'mcr.microsoft.com/mssql/server:2022-latest'
        CONTAINER_NAME = 'nah'
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    sh "git clone https://github.com/Vlad1ck228/cloud.git"
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    // Build frontend Docker image
                    sh "docker build -t ${FRONTEND_IMAGE_NAME} /home/ubuntu/cloud/FrontEnd/my-app/"
                    
                    // Navigate to the backend directory
                    dir("/home/ubuntu/cloud/BackEnd/Amazon-clone/") {
                        // Build backend Docker image
                        sh "docker build -t ${BACKEND_IMAGE_NAME} ."
                    }
                }
            }
        }

        stage('Run Docker Containers') {
            steps {
                script {
                    // Run database Docker container
                    sh "docker run -e \"ACCEPT_EULA=Y\" -e \"MSSQL_SA_PASSWORD=Qwerty-1\" -p 1433:1433 --name db_container -d ${DB_IMAGE_NAME}"
                    sh "docker run --name=frontend_container -d -p 81:80 ${FRONTEND_IMAGE_NAME}"
                    sh "docker run --name=backend_container -d -p 5034:5034 ${BACKEND_IMAGE_NAME}"
                }
            }
        }

        stage('Check Container Status') {
            steps {
                script {
                    def dbStatus = sh(script: "docker inspect -f '{{.State.Status}}' db_container", returnStdout: true).trim()
                    def frontendStatus = sh(script: "docker inspect -f '{{.State.Status}}' frontend_container", returnStdout: true).trim()
                    def backendStatus = sh(script: "docker inspect -f '{{.State.Status}}' backend_container", returnStdout: true).trim()

                    if (dbStatus == 'running' && frontendStatus == 'running' && backendStatus == 'running') {
                        echo "All containers are running successfully!"
                    } else {
                        error "One or more containers failed to start!"
                    }
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    // Push frontend Docker image to Docker Hub
                    sh "docker push ${FRONTEND_IMAGE_NAME}"
                    // Push backend Docker image to Docker Hub
                    sh "docker push ${BACKEND_IMAGE_NAME}"
                }
            }
        }
    }

    post {
        success {
            script {
                // Remove all containers if the build succeeds
                sh "docker rm -f db_container frontend_container backend_container"
            }
        }
        failure {
            script {
                // Remove all containers if the build fails
                sh "docker rm -f db_container frontend_container backend_container"
            }
        }
    }
}

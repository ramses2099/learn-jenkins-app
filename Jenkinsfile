pipeline {
    agent any

    stages {
        stage('Build') {

            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                     ls -la
                     node --version
                     npm --version
                     npm ci
                     npm run build                
                '''
            }       
        }
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    test -f build/index.html
                    npm test
                '''
            }  

        }
        stage('Deploy') {
            // Environment variables for easier management
            environment {
                DOCKER_IMAGE_NAME = 'my-local-node-app'
                // Use a simple tag like 'latest' for a local run, or the build number
                IMAGE_TAG = 'latest' 
                CONTAINER_NAME = 'my-local-app-container'
            }
            steps {
                script {
                    def imageFullName = "${DOCKER_IMAGE_NAME}:${IMAGE_TAG}"

                    echo "Building Docker image: ${imageFullName}"
                    // 1. Build the Docker Image
                    // Uses the Dockerfile found in the workspace (containing the 'build/' artifacts)
                    sh "docker build -t ${imageFullName} ."

                    echo "Stopping and removing any old container named: ${CONTAINER_NAME}"
                    // 2. Stop and Remove Previous Container (if it exists)
                    // The '|| true' ensures the pipeline doesn't fail if the container doesn't exist.
                    sh "docker stop ${CONTAINER_NAME} || true"
                    sh "docker rm ${CONTAINER_NAME} || true"

                    echo "Running new container on port 8080"
                    // 3. Run the New Container
                    // -d: detached mode (run in background)
                    // --name: assigns a specific name
                    // -p: maps the host port 8080 to the container's internal port 80
                    sh "docker run -d --name ${CONTAINER_NAME} -p 8080:80 ${imageFullName}"
                    
                    echo "Deployment to local Docker container successful! Access via http://localhost:8080"
                }
            }
        }
    }
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}

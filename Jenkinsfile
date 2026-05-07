pipeline {
    environment {
        DOCKER_ID = 'nimamrze'
        DOCKER_IMAGE_CAST = 'castservice'
        DOCKER_IMAGE_MOVIE = 'movieservice'
        DOCKER_IMAGE_NGINX = 'nginx:latest'
        DOCKER_IMAGE_DB = 'postgres:12.1-alpine'
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    sh '''
                    echo "Building the application..."
                    docker rm -f cast || true
                    docker rm -f movie || true
                    docker build -t ${DOCKER_ID}/${DOCKER_IMAGE_MOVIE}:${DOCKER_TAG} ./movie-service
                    docker build -t ${DOCKER_ID}/${DOCKER_IMAGE_CAST}:${DOCKER_TAG} ./cast-service
                    sleep 5
                    '''
                }
            }
        }

        stage('Run') {
            steps {
                script {
                    sh '''
                    echo "Running the application..."
                    docker run -d --name movie -p 8080:8080 ${DOCKER_ID}/${DOCKER_IMAGE_MOVIE}:${DOCKER_TAG}
                    docker run -d --name cast -p 8081:8081 ${DOCKER_ID}/${DOCKER_IMAGE_CAST}:${DOCKER_TAG}
                    sleep 5
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                sh 'echo "Running tests..."'
            // Add your test commands here
            }
        }

        stage('Docker Build') {
            steps {
                sh 'echo "Building Docker image..."'
                sh "docker build -t ${DOCKER_ID}/${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage('Docker Push') {
            steps {
                sh 'echo "Pushing Docker image to registry..."'
                sh "docker push ${DOCKER_ID}/${DOCKER_IMAGE}:${DOCKER_TAG}"
            }
        }
    }
}

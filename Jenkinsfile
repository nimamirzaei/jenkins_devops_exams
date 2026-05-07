pipeline {
    agent any

    parameters {
        choice(name: 'ACTION', choices: ['Deploy', 'Stop'], description: 'Deploy or Stop environment')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'staging', 'prod', 'all'], description: 'Target environment')
    }

    environment {
        DOCKER_ID = 'nimamrze'
        DOCKER_TAG = "v.${BUILD_NUMBER}.0"

        MOVIE_IMAGE = 'movie-service'
        CAST_IMAGE = 'cast-service'

        CHART_PATH = './movie-app-chart'
    }

    stages {
        stage('Checkout') {
            when {
                expression { params.ACTION == 'Deploy' }
            }
            steps {
                checkout scm
            }
        }

        stage('Build Images') {
            when {
                expression { params.ACTION == 'Deploy' }
            }
            steps {
                sh '''
                docker build -t $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG ./movie-service
                docker build -t $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG ./cast-service
                '''
            }
        }

        stage('Docker Login') {
            when {
                expression { params.ACTION == 'Deploy' }
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'DOCKER_HUB_PASS',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Images') {
            when {
                expression { params.ACTION == 'Deploy' }
            }
            steps {
                sh '''
                docker push $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG
                docker push $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG
                '''
            }
        }

        stage('Deploy Dev') {
            when {
                expression { params.ACTION == 'Deploy' && (params.ENVIRONMENT == 'dev' || params.ENVIRONMENT == 'all') }
            }
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                sh '''
                input message: 'Deploy to Dev?'
                rm -Rf .kube
                mkdir .kube
                cat $KUBECONFIG > .kube/config
                helm upgrade --install movie-app-dev $CHART_PATH \
                  -n dev \
                  --create-namespace \
                  --set image.tag=$DOCKER_TAG \
                  --set nginx.nodePort=30080
                '''
            }
        }

        stage('Deploy QA') {
            when {
                expression { params.ACTION == 'Deploy' && (params.ENVIRONMENT == 'qa' || params.ENVIRONMENT == 'all') }
            }
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                sh '''
                rm -Rf .kube
                mkdir .kube
                cat $KUBECONFIG > .kube/config
                helm upgrade --install movie-app-qa $CHART_PATH \
                  -n qa \
                  --create-namespace \
                  --set image.tag=$DOCKER_TAG \
                  --set nginx.nodePort=30081
                '''
            }
        }

        stage('Deploy Staging') {
            when {
                expression { params.ACTION == 'Deploy' && (params.ENVIRONMENT == 'staging' || params.ENVIRONMENT == 'all') }
            }
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                sh '''
                rm -Rf .kube
                mkdir .kube
                cat $KUBECONFIG > .kube/config
                helm upgrade --install movie-app-staging $CHART_PATH \
                  -n staging \
                  --create-namespace \
                  --set image.tag=$DOCKER_TAG \
                  --set nginx.nodePort=30082
                '''
            }
        }

        stage('Deploy Prod') {
            when {
                expression { params.ACTION == 'Deploy' && (params.ENVIRONMENT == 'prod' || params.ENVIRONMENT == 'all') }
            }
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                input message: 'Deploy to production?'
                sh '''
                rm -Rf .kube
                mkdir .kube
                cat $KUBECONFIG > .kube/config
                helm upgrade --install movie-app-prod $CHART_PATH \
                  -n prod \
                  --create-namespace \
                  --set image.tag=$DOCKER_TAG \
                  --set nginx.nodePort=30083
                '''
            }
        }

        stage('Stop Dev') {
            when {
                expression { params.ACTION == 'Stop' && (params.ENVIRONMENT == 'dev' || params.ENVIRONMENT == 'all') }
            }
            steps {
                input message: 'Stop DEV environment?'
                sh '''
                helm uninstall movie-app-dev -n dev || true
                '''
            }
        }

        stage('Stop QA') {
            when {
                expression { params.ACTION == 'Stop' && (params.ENVIRONMENT == 'qa' || params.ENVIRONMENT == 'all') }
            }
            steps {
                input message: 'Stop QA environment?'
                sh '''
                helm uninstall movie-app-qa -n qa || true
                '''
            }
        }

        stage('Stop Staging') {
            when {
                expression { params.ACTION == 'Stop' && (params.ENVIRONMENT == 'staging' || params.ENVIRONMENT == 'all') }
            }
            steps {
                input message: 'Stop STAGING environment?'
                sh '''
                helm uninstall movie-app-staging -n staging || true
                '''
            }
        }

        stage('Stop Prod') {
            when {
                expression { params.ACTION == 'Stop' && (params.ENVIRONMENT == 'prod' || params.ENVIRONMENT == 'all') }
            }
            steps {
                input message: 'Stop PRODUCTION environment?'
                sh '''
                helm uninstall movie-app-prod -n prod || true
                '''
            }
        }
    }

    post {
        always {
            sh '''
            echo "===== DEV ====="
            kubectl get pods -n dev || true
            kubectl get svc -n dev || true

            echo "===== QA ====="
            kubectl get pods -n qa || true
            kubectl get svc -n qa || true

            echo "===== STAGING ====="
            kubectl get pods -n staging || true
            kubectl get svc -n staging || true

            echo "===== PROD ====="
            kubectl get pods -n prod || true
            kubectl get svc -n prod || true
            '''
        }
    }
}

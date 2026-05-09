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

        CHART_PATH = './charts'

        NS_DEV = 'jenkins-dev'
        NS_QA = 'jenkins-qa'
        NS_STAGING = 'jenkins-staging'
        NS_PROD = 'jenkins-prod'

        HOST_DEV = 'movie-dev.services-cloud.abrdns.com'
        HOST_QA = 'movie-qa.services-cloud.abrdns.com'
        HOST_STAGING = 'movie-staging.services-cloud.abrdns.com'
        HOST_PROD = 'movie-prod.services-cloud.abrdns.com'
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
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    mkdir -p ~/.kube
                    cp "$KUBECONFIG_FILE" ~/.kube/config
                    chmod 600 ~/.kube/config

                    helm upgrade --install movie-app-dev $CHART_PATH \
                      -n $NS_DEV \
                      --create-namespace \
                      --set image.tag=$DOCKER_TAG \
                      --set ingress.enabled=true \
                      --set ingress.host=$HOST_DEV
                    '''
                }
            }
        }

        stage('Deploy QA') {
            when {
                expression { params.ACTION == 'Deploy' && (params.ENVIRONMENT == 'qa' || params.ENVIRONMENT == 'all') }
            }
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    mkdir -p ~/.kube
                    cp "$KUBECONFIG_FILE" ~/.kube/config
                    chmod 600 ~/.kube/config

                    helm upgrade --install movie-app-qa $CHART_PATH \
                      -n $NS_QA \
                      --create-namespace \
                      --set image.tag=$DOCKER_TAG \
                      --set ingress.enabled=true \
                      --set ingress.host=$HOST_QA
                    '''
                }
            }
        }

        stage('Deploy Staging') {
            when {
                expression { params.ACTION == 'Deploy' && (params.ENVIRONMENT == 'staging' || params.ENVIRONMENT == 'all') }
            }
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    mkdir -p ~/.kube
                    cp "$KUBECONFIG_FILE" ~/.kube/config
                    chmod 600 ~/.kube/config

                    helm upgrade --install movie-app-staging $CHART_PATH \
                      -n $NS_STAGING \
                      --create-namespace \
                      --set image.tag=$DOCKER_TAG \
                      --set ingress.enabled=true \
                      --set ingress.host=$HOST_STAGING
                    '''
                }
            }
        }

        stage('Deploy Prod') {
            when {
                expression { params.ACTION == 'Deploy' && (params.ENVIRONMENT == 'prod' || params.ENVIRONMENT == 'all') }
            }
            steps {
                input message: 'Deploy to production?'
        
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    mkdir -p ~/.kube
                    cp "$KUBECONFIG_FILE" ~/.kube/config
                    chmod 600 ~/.kube/config
        
                    helm upgrade --install movie-app-prod $CHART_PATH \
                      -n $NS_PROD \
                      --create-namespace \
                      --set image.tag=$DOCKER_TAG \
                      --set ingress.enabled=true \
                      --set ingress.host=$HOST_PROD
                    '''
                }
            }
        }

        stage('Stop Dev') {
            when {
                expression { params.ACTION == 'Stop' && (params.ENVIRONMENT == 'dev' || params.ENVIRONMENT == 'all') }
            }
            steps {
                input message: 'Stop DEV environment?'

                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    mkdir -p ~/.kube
                    cp "$KUBECONFIG_FILE" ~/.kube/config
                    chmod 600 ~/.kube/config

                    helm uninstall movie-app-dev -n $NS_DEV || true
                    '''
                }
            }
        }

        stage('Stop QA') {
            when {
                expression { params.ACTION == 'Stop' && (params.ENVIRONMENT == 'qa' || params.ENVIRONMENT == 'all') }
            }
            steps {
                input message: 'Stop QA environment?'

                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    mkdir -p ~/.kube
                    cp "$KUBECONFIG_FILE" ~/.kube/config
                    chmod 600 ~/.kube/config

                    helm uninstall movie-app-qa -n $NS_QA || true
                    '''
                }
            }
        }

        stage('Stop Staging') {
            when {
                expression { params.ACTION == 'Stop' && (params.ENVIRONMENT == 'staging' || params.ENVIRONMENT == 'all') }
            }
            steps {
                input message: 'Stop STAGING environment?'

                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    mkdir -p ~/.kube
                    cp "$KUBECONFIG_FILE" ~/.kube/config
                    chmod 600 ~/.kube/config

                    helm uninstall movie-app-staging -n $NS_STAGING || true
                    '''
                }
            }
        }

        stage('Stop Prod') {
            when {
                expression { params.ACTION == 'Stop' && (params.ENVIRONMENT == 'prod' || params.ENVIRONMENT == 'all') }
            }
            steps {
                input message: 'Stop PRODUCTION environment?'

                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    mkdir -p ~/.kube
                    cp "$KUBECONFIG_FILE" ~/.kube/config
                    chmod 600 ~/.kube/config

                    helm uninstall movie-app-prod -n $NS_PROD || true
                    '''
                }
            }
        }
    }

    post {
        always {
            withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                sh '''
                mkdir -p ~/.kube
                cp "$KUBECONFIG_FILE" ~/.kube/config
                chmod 600 ~/.kube/config

                echo "===== HELM RELEASES ====="
                helm list -A || true

                echo "===== JENKINS DEV ====="
                kubectl get pods -n $NS_DEV || true
                kubectl get svc -n $NS_DEV || true
                kubectl get ingress -n $NS_DEV || true

                echo "===== JENKINS QA ====="
                kubectl get pods -n $NS_QA || true
                kubectl get svc -n $NS_QA || true
                kubectl get ingress -n $NS_QA || true

                echo "===== JENKINS STAGING ====="
                kubectl get pods -n $NS_STAGING || true
                kubectl get svc -n $NS_STAGING || true
                kubectl get ingress -n $NS_STAGING || true

                echo "===== JENKINS PROD ====="
                kubectl get pods -n $NS_PROD || true
                kubectl get svc -n $NS_PROD || true
                kubectl get ingress -n $NS_PROD || true
                '''
            }
        }
    }
}
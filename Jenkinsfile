pipeline {
    agent any
    
    environment {
        DOCKER_ID = "aminamr"
        MOVIE_IMAGE = "movie-service"
        CAST_IMAGE = "cast-service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    
    stages {
        stage('Docker Build') {
            steps {
                script {
                    sh '''
                    docker build -t $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG ./movie-service
                    docker build -t $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG ./cast-service
                    '''
                }
            }
        }
        
        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG
                    docker push $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        
        stage('Deploy to Dev') {
            environment {
                KUBECONFIG_CONTENT = credentials("KUBECONFIG")
            }
            steps {
                script {
                    sh '''
                    rm -rf .kube
                    mkdir -p .kube
                    cat $KUBECONFIG_CONTENT > .kube/config
                    chmod 600 .kube/config
                    export KUBECONFIG=$(pwd)/.kube/config
                    cp charts/values.yaml values-dev.yaml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values-dev.yaml
                    sed -i "s+repository.*+repository: ${DOCKER_ID}/${MOVIE_IMAGE}+g" values-dev.yaml
                    helm upgrade --install movie-cast-app charts --values=values-dev.yaml --namespace dev
                    '''
                }
            }
        }
        
        stage('Deploy to QA') {
            environment {
                KUBECONFIG_CONTENT = credentials("KUBECONFIG")
            }
            steps {
                script {
                    sh '''
                    rm -rf .kube
                    mkdir -p .kube
                    cat $KUBECONFIG_CONTENT > .kube/config
                    chmod 600 .kube/config
                    export KUBECONFIG=$(pwd)/.kube/config
                    cp charts/values.yaml values-qa.yaml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values-qa.yaml
                    sed -i "s+repository.*+repository: ${DOCKER_ID}/${MOVIE_IMAGE}+g" values-qa.yaml
                    helm upgrade --install movie-cast-app charts --values=values-qa.yaml --namespace qa
                    '''
                }
            }
        }
        
        stage('Deploy to Staging') {
            environment {
                KUBECONFIG_CONTENT = credentials("KUBECONFIG")
            }
            steps {
                script {
                    sh '''
                    rm -rf .kube
                    mkdir -p .kube
                    cat $KUBECONFIG_CONTENT > .kube/config
                    chmod 600 .kube/config
                    export KUBECONFIG=$(pwd)/.kube/config
                    cp charts/values.yaml values-staging.yaml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values-staging.yaml
                    sed -i "s+repository.*+repository: ${DOCKER_ID}/${MOVIE_IMAGE}+g" values-staging.yaml
                    helm upgrade --install movie-cast-app charts --values=values-staging.yaml --namespace staging
                    '''
                }
            }
        }
        
        stage('Deploy to Prod') {
            when {
                branch 'master'
            }
            environment {
                KUBECONFIG_CONTENT = credentials("KUBECONFIG")
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy to production?', ok: 'Yes'
                }
                script {
                    sh '''
                    rm -rf .kube
                    mkdir -p .kube
                    cat $KUBECONFIG_CONTENT > .kube/config
                    chmod 600 .kube/config
                    export KUBECONFIG=$(pwd)/.kube/config
                    cp charts/values.yaml values-prod.yaml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values-prod.yaml
                    sed -i "s+repository.*+repository: ${DOCKER_ID}/${MOVIE_IMAGE}+g" values-prod.yaml
                    helm upgrade --install movie-cast-app charts --values=values-prod.yaml --namespace prod
                    '''
                }
            }
        }
    }
}
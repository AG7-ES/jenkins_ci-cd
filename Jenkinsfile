pipeline {
    environment { // Déclaration des variables d'environnement
        DOCKER_ID = "ag7es" // Remplacez par votre docker-id
        DOCKER_IMAGE_MOVIE = "movie-image"
        DOCKER_IMAGE_CAST = "cast-image"
        DOCKER_TAG = "v.${BUILD_ID}.0" // Tag de version avec l'ID de build
    }
    agent any // Jenkins peut sélectionner tous les agents disponibles

    stages {
        stage('Build, Run and Test in Parallel') {
            parallel {
                stage('Movie Service Pipeline') {
                    stages {
                        stage('Docker Build Movie Image') {
                            steps {
                                script {
                                    sh '''
                                    docker rm -f jenkins || true
                                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG ./movie-service/
                                    sleep 6
                                    '''
                                }
                            }
                        }
                        stage('Docker Run Movie Image') {
                            steps {
                                script {
                                    sh '''
			            docker rm -f movie_service || true
                                    docker run -d -p 8080:80 --name movie_service $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                                    sleep 10
                                    '''
                                }
                            }
                        }
                        stage('Test Movie Service') {
                            steps {
                                script {
                                    sh '''
                                    curl localhost
                                    '''
                                }
                            }
                        }
                    }
                }
                stage('Cast Service Pipeline') {
                    stages {
                        stage('Docker Build Cast Image') {
                            steps {
                                script {
                                    sh '''
                                    docker rm -f jenkins || true
                                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG ./cast-service/
                                    sleep 6
                                    '''
                                }
                            }
                        }
                        stage('Docker Run Cast Image') {
                            steps {
                                script {
                                    sh '''
 			            docker rm -f cast_service || true
                                    docker run -d -p 8081:80 --name cast_service $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                                    sleep 10
                                    '''
                                }
                            }
                        }
                        stage('Test Cast Service') {
                            steps {
                                script {
                                    sh '''
                                    curl localhost
                                    '''
                                }
                            }
                        }
                    }
                }
            }
        }
        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // Récupère le mot de passe Docker depuis Jenkins credentials
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                    docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploiement en dev') {
            environment {
                KUBECONFIG = credentials("config") // Récupère kubeconfig depuis Jenkins credentials
            }
            steps {
                script {
                    sh '''
		            rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app ./charts --values=values.yml --namespace dev
                    '''
                }
            }
        }
        stage('Deploiement en QA') {
            environment {
                KUBECONFIG = credentials("config") // Récupère kubeconfig depuis Jenkins credentials
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app ./charts --values=values.yml --namespace qa
                    '''
                }
            }
        }
        stage('Deploiement en staging') {
            environment {
                KUBECONFIG = credentials("config") // Récupère kubeconfig depuis Jenkins credentials
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app ./charts --values=values.yml --namespace staging
                    '''
                }
            }
        }
        stage('Deploiement en prod') {
            environment {
                KUBECONFIG = credentials("config") // Récupère kubeconfig depuis Jenkins credentials
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app ./charts --values=values.yml --namespace prod
                    '''
                }
            }
        }
    }
}
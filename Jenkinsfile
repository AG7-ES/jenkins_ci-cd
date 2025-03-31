pipeline {

    environment {
        DOCKER_ID = "ag7es"
        MOVIE_SERVICE_IMAGE = "movie_service"
        CAST_SERVICE_IMAGE = "cast_service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
        MOVIE_IMAGE_NAME = "${DOCKER_ID}/${MOVIE_SERVICE_IMAGE}:${DOCKER_TAG}"
        CAST_IMAGE_NAME = "${DOCKER_ID}/${CAST_SERVICE_IMAGE}:${DOCKER_TAG}"
    }

    agent any

    stages {

        stage('Docker Build') {
            steps {
                script {
                    sh '''
                    docker rm -f jenkins
                    docker build -t $CAST_IMAGE_NAME ./cast-service
                    sleep 6
                    '''
                }
                script {
                    sh '''
                    docker build -t $MOVIE_IMAGE_NAME ./movie-service
                    sleep 6
                    '''
                }
            }
        }

        stage('Docker Run') {
            steps {
                script {
                    sh '''
                    docker run -d --rm -p 80:8000 $CAST_IMAGE_NAME
                    sleep 10
                    '''
                }
                script {
                    sh '''
                    docker run -d --rm -p 80:8000 $MOVIE_IMAGE_NAME
                    sleep 10
                    '''
                }
            }
        }

        stage('Test Acceptance') {
            steps {
                script {
                    sh '''
                    curl http://localhost:8080/api/v1/casts/docs
                    '''
                }
                script {
                    sh '''
                    curl http://localhost:8080/api/v1/movies/docs
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'DOCKER_HUB_CREDENTIALS', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $CAST_IMAGE_NAME
                        docker push $MOVIE_IMAGE_NAME
                        '''
                    }
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                        sh '''
                        rm -Rf .kube
                        mkdir .kube
                        ls
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yaml
                        cat values.yaml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                        helm upgrade --install app charts --values=values.yaml --namespace dev
                        '''
                    }
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                        sh '''
                        rm -Rf .kube
                        mkdir .kube
                        ls
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yaml
                        cat values.yaml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                        helm upgrade --install app charts --values=values.yaml --namespace qa
                        '''
                    }
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                        sh '''
                        rm -Rf .kube
                        mkdir .kube
                        ls
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yaml
                        cat values.yaml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                        helm upgrade --install app charts --values=values.yaml --namespace staging
                        '''
                    }
                }
            }
        }

        stage('Deploy to Prod') {
            when {
                branch 'master'
            }
            steps {
                script {
                    try {
                        timeout(time: 15, unit: 'MINUTES') {
                            def userInput = input(
                                message: 'Do you want to deploy in production?',
                                parameters: [choice(name: 'Deploy', choices: 'Yes\nAbort', description: 'Select Yes to continue or Abort to stop')]
                            )
                            if (userInput == 'Yes') {
                                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                                    sh '''
                                    rm -Rf .kube
                                    mkdir .kube
                                    ls
                                    cat $KUBECONFIG > .kube/config
                                    cp charts/values.yaml values.yaml
                                    cat values.yaml
                                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                                    helm upgrade --install app charts --values=values.yaml --namespace prod
                                    '''
                                }
                            } else {
                                error "Deployment aborted by admin!"
                            }
                        }
                    } catch (err) {
                        error "Deployment timed out or aborted: ${err}"
                    }
                }
            }
        }


    }
}

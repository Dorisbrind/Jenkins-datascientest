pipeline {
    agent any

    environment {
        DOCKER_ID = "docker.io/dorismb"   // Ton compte DockerHub
        IMAGE_TAG = "${BUILD_NUMBER}"    // Numéro de build Jenkins comme tag
        APP_NAME = "app"                 // Nom de ton application (utilisé dans helm)
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    def services = ["movie-service", "cast-service"]
                    services.each { svc ->
                        sh "docker build -t ${svc}:latest ${svc}/"
                    }
                }
            }
        }

        stage('Push Docker Images') {
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    def services = ["movie-service", "cast-service"]
                    services.each { svc ->
                        sh """
                            echo "$DOCKER_PASS" | docker login -u dorismb --password-stdin
                            docker tag ${svc}:latest dorismb/${svc}:${IMAGE_TAG}
                            docker push dorismb/${svc}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('Deploiement en dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh """
                        # creé un namespace dev
                        kubectl get namespace dev || kubectl create namespace dev
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yml
                        
                        helm upgrade --install ${APP_NAME}-dev charts \
                          --namespace dev \
                          --set image.cat-service.tag=${IMAGE_TAG} \
                          --set image.movie-service.tag=${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploiement en QA') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh """
                         # creé un namespace QA
                        kubectl get namespace qa || kubectl create namespace qa
                        rm -Rf .kube
                        mkdir .kube
                        cat \$KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yml

                        helm upgrade --install ${APP_NAME}-qa charts \
                          --namespace qa \
                          --set image.cat-service.tag=${IMAGE_TAG} \
                          --set image.movie-service.tag=${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploiement en staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh """
                        # creé un namespace staging
                        kubectl get namespace staging || kubectl create namespace staging
                        rm -Rf .kube
                        mkdir .kube
                        cat \$KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yml
                        
                        helm upgrade --install ${APP_NAME}-staging charts \
                          --namespace staging \
                          --set image.cat-service.tag=${IMAGE_TAG} \
                          --set image.movie-service.tag=${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploiement en prod') {
            when {
                branch "master"
            }
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                input message: 'voulez-vous déployer en production ?', ok: 'oui'
                script {
                    sh """
                        # creé un namespace prod
                        kubectl get namespace prod || kubectl create namespace prod
                        rm -Rf .kube
                        mkdir .kube
                        cat \$KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yml
                        
                        helm upgrade --install ${APP_NAME}-prod charts \
                          --namespace prod \
                          --set image.cat-service.tag=${IMAGE_TAG} \
                          --set image.movie-service.tag=${IMAGE_TAG}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline terminée avec succès"
        }
        failure {
            echo "❌ La pipeline a échoué"
        }
    }
}

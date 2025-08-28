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
                            docker login -u $DOCKER_ID -p $DOCKER_PASS
                            docker tag ${svc}:latest $REGISTRY/${svc}:$IMAGE_TAG
                            docker push $DOCKER_ID/${svc}:$IMAGE_TAG
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
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yml
                        sed -i "s+movie-service:.*+movie-service: ${IMAGE_TAG}+g" values.yml
                        sed -i "s+cast-service:.*+cast-service: ${IMAGE_TAG}+g" values.yml
                        helm upgrade --install ${APP_NAME}-dev charts --values=values.yml --namespace dev
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
                        rm -Rf .kube
                        mkdir .kube
                        cat \$KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yml
                        sed -i "s+movie-service:.*+movie-service: ${IMAGE_TAG}+g" values.yml
                        sed -i "s+cast-service:.*+cast-service: ${IMAGE_TAG}+g" values.yml
                        helm upgrade --install ${APP_NAME}-qa charts --values=values.yml --namespace qa
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
                        rm -Rf .kube
                        mkdir .kube
                        cat \$KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yml
                        sed -i "s+movie-service:.*+movie-service: ${IMAGE_TAG}+g" values.yml
                        sed -i "s+cast-service:.*+cast-service: ${IMAGE_TAG}+g" values.yml
                        helm upgrade --install ${APP_NAME}-staging charts --values=values.yml --namespace staging
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
                input message: 'Voulez-vous déployer en production ?', ok: 'Oui'
                script {
                    sh """
                        rm -Rf .kube
                        mkdir .kube
                        cat \$KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yml
                        sed -i "s+movie-service:.*+movie-service: ${IMAGE_TAG}+g" values.yml
                        sed -i "s+cast-service:.*+cast-service: ${IMAGE_TAG}+g" values.yml
                        helm upgrade --install ${APP_NAME}-prod charts --values=values.yml --namespace prod
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

pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/dorismb"
        APP_NAME = "app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
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
                    sh 'docker-compose build'
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    def services = ["movie-service", "cast-service"]
                    services.each { svc ->
                        sh """
                            docker tag ${svc}:latest $REGISTRY/$svc:$IMAGE_TAG
                            docker push $REGISTRY/$svc:$IMAGE_TAG
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
                        cat \$KUBECONFIG > .kube/config
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
                branch "main"
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


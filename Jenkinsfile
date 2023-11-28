pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_REPO = 'bashidkk/my-node-app'
        APP_NAME = 'my-app'
        KUBE_NAMESPACE = 'default'
        HELM_CHART_NAME = 'node-chart'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build and push Docker image
                    docker.build("${DOCKER_REGISTRY}/${DOCKER_REPO}/${APP_NAME}:${BUILD_NUMBER}").push()
                }
            }
        }

        stage('Update Helm Values') {
            steps {
                script {
                    // Update the image tag in the values.yaml file of your Helm chart
                    sh "sed -i 's|imageTag:.*|imageTag: ${BUILD_NUMBER}|' ${WORKSPACE}/node-chart/values.yaml"
                }
            }
        }

        stage('Zip Helm Chart') {
            steps {
                script {
                    // Zip the Helm chart
                    sh "cd ${WORKSPACE} && tar -czf ${WORKSPACE}/${APP_NAME}-${BUILD_NUMBER}.tgz ."
                }
            }
        }

        stage('Push Helm Chart to Registry') {
            steps {
                script {
                    // Push the Helm chart to a registry
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker_cred') {
                        def dockerImage = docker.image("${DOCKER_REGISTRY}/${DOCKER_REPO}/${APP_NAME}:${BUILD_NUMBER}")
                        dockerImage.push()
                        sh "docker tag ${DOCKER_REGISTRY}/${DOCKER_REPO}/${APP_NAME}:${BUILD_NUMBER} ${DOCKER_REGISTRY}/${DOCKER_REPO}/${APP_NAME}:latest"
                        sh "docker push ${DOCKER_REGISTRY}/${DOCKER_REPO}/${APP_NAME}:latest"
                    }
                }
            }
        }
    }
}

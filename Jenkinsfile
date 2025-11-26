pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = credentials('dockerhub-credentials')  // Jenkins credential ID
        IMAGE_NAME      = "ajaxr7781/ci-cd-demo"            // Replace with your DockerHub username
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${env.BUILD_NUMBER} .
                    docker tag ${IMAGE_NAME}:${env.BUILD_NUMBER} ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh """
                    echo "${DOCKERHUB_CREDS_PSW}" | docker login -u "${DOCKERHUB_CREDS_USR}" --password-stdin

                    docker push ${IMAGE_NAME}:${env.BUILD_NUMBER}
                    docker push ${IMAGE_NAME}:latest

                    docker logout
                """
            }
        }

        stage('Deploy to k3s via Ansible') {
            steps {
                sh """
                    ansible -i ansible/inventory k8s \
                        -e image_tag=${env.BUILD_NUMBER} \
                        -m ping

                    ansible -i ansible/inventory k8s \
                        -b -vvv \
                        -m debug -a 'msg="Starting deployment..."'

                    ansible -i ansible/inventory k8s \
                        -b \
                        -m shell -a 'kubectl get nodes'

                    ansible-playbook -i ansible/inventory ansible/deploy_k8s.yml
                """
            }
        }
    }

    post {
        always {
            echo "Build completed: ${currentBuild.currentResult}"
        }
    }
}

pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-hub'
        DOCKER_IMAGE_NAME = 'flask'
        DOCKER_IMAGE_TAG = '0.1'
        TRIVY_VERSION = '0.36.0'
    }

    stages {
        stage('Setup') {
            steps {
                script {
                    sh 'pip install bandit safety pip-audit'
                }
            }
        }

        stage('Security Tests') {
            steps {
                script {
                    sh 'bandit -r simple_flask_app'
                    sh 'safety check'
                    sh 'pip-audit'
                }
            }
        }

        stage('Docker Image Build') {
            steps {
                script {
                    dir('simple_flask_app') {
                        sh "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
                    }
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    sh "curl -sSfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -"
                    sh "trivy image ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                        sh 'docker login --username $USERNAME --password $PASSWORD'
                        sh "docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} $USERNAME/flask-app:${DOCKER_IMAGE_TAG}"
                        sh "docker push $USERNAME/flask-app:${DOCKER_IMAGE_TAG}"
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}




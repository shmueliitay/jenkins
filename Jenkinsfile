pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'itay80'
        DOCKERHUB_REPO = 'jenkins-work'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/shmueliitay/jenkins.git'
            }
        }

        stage('Debug Permissions') { // <--- ADD THIS NEW STAGE
            steps {
                script {
                    sh 'whoami' // Show the user running the command
                    sh 'groups' // Show the groups the current user belongs to
                    sh 'ls -l /var/run/docker.sock' // Show permissions of the Docker socket
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKERHUB_USER_VAR', passwordVariable: 'DOCKERHUB_PASSWORD_VAR')]) {
                        sh "echo ${DOCKERHUB_PASSWORD_VAR} | docker login -u ${DOCKERHUB_USER_VAR} --password-stdin"
                    }
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    sh "docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${IMAGE_TAG}"
                }
            }
        }
    }
}

pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'itay80'
        DOCKERHUB_PASSWORD = credentials('docker')
        DOCKERHUB_REPO = 'jenkins-work'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/shmueliitay/jenkins.git'
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
                sh '''
                echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
                '''
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

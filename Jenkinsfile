// Jenkinsfile
pipeline {
    agent any

    // Define environment variables for the Docker registry and image name.
    // IMPORTANT: Replace 'your-dockerhub-username' with your actual Docker Hub username
    // or the URL of your private registry.
    environment {
        DOCKER_REGISTRY = 'your-dockerhub-username' // e.g., 'myusername' or 'registry.example.com/myusername'
        DOCKER_IMAGE_NAME = 'my-docker-app'
        // DOCKER_CREDENTIALS_ID: This is the ID of your 'Username with password' credential in Jenkins.
        // You need to create this credential in Jenkins (Dashboard -> Manage Jenkins -> Manage Credentials -> System -> Global credentials (unrestricted) -> Add Credentials).
        // Set Kind to 'Username with password', provide your Docker Hub username and password, and give it an ID like 'dockerhub-credentials'.
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
    }

    // Triggers define what events will start this pipeline.
    // This pipeline will be triggered by pushes to the 'main' branch on GitHub.
    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                // This step assumes your Jenkins job is configured to clone the repository.
                // If not, you would typically add a 'git' step here:
                // git branch: 'main', url: 'https://github.com/your-username/my-docker-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:latest"
                // Build the Docker image.
                // The tag includes the registry username/URL for pushing.
                sh "docker build -t ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:latest ."
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Pushing Docker image: ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:latest to registry..."
                // Use withCredentials to securely access Docker Hub credentials.
                // The username and password will be available as environment variables within this block.
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    // Log in to Docker Hub using the credentials.
                    sh "echo \"${DOCKER_PASSWORD}\" | docker login -u \"${DOCKER_USERNAME}\" --password-stdin ${DOCKER_REGISTRY.split('/')[0]}" // Handles cases like 'username' or 'registry.example.com/username'
                    // Push the Docker image to the registry.
                    sh "docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:latest"
                    // Log out from Docker Hub (good practice for security).
                    sh "docker logout ${DOCKER_REGISTRY.split('/')[0]}"
                }
            }
        }

        stage('Run Docker Container (Optional - for verification)') {
            steps {
                echo 'Running Docker container locally for verification...'
                // This step is optional but good for verifying the built image works.
                // The '-rm' flag ensures the container is removed after it exits.
                sh "docker run --rm ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:latest"
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished. Cleaning up workspace...'
            // Clean the Jenkins workspace after the pipeline completes.
            cleanWs()
        }
        success {
            echo 'Docker app pipeline succeeded! 🎉 Image built and pushed.'
            // Add notifications here, e.g., Slack, email
            // slackSend channel: '#dev-notifications', message: "Build successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}. Image pushed to ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:latest"
        }
        failure {
            echo 'Docker app pipeline failed! ❌'
            // Add failure notifications
            // slackSend channel: '#dev-notifications', message: "Build failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
    }
}

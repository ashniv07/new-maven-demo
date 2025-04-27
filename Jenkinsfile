pipeline {
    agent any

    environment {
        IMAGE_NAME = "3.142.249.69:5000/backend-app" // Nexus Private Registry
        IMAGE_TAG = "v${BUILD_NUMBER}"
        FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"

        APP_REPO_URL = "https://github.com/SUBASHREE-KB/applicationcode.git"
        MANIFEST_REPO_URL = "https://github.com/SUBASHREE-KB/manifests.git"
        MANIFEST_REPO_BRANCH = "main"
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                git url: "${APP_REPO_URL}", branch: 'main'
            }
        }

        stage('Build and Push Docker Image') {
            environment {
                DOCKER_CREDS = credentials('nexus-docker-credentials')  // Use Jenkins credentials to access Docker registry
            }
            steps {
                script {
                    // Login to Nexus Docker registry and build & push image
                    docker.withRegistry('http://3.142.249.69:5000', 'nexus-docker-credentials') {
                        def customImage = docker.build(FULL_IMAGE)  // Build the Docker image
                        customImage.push()  // Push the image to Nexus
                    }
                }
            }
        }

        stage('Clone Manifest Repo and Update Image Tag') {
            environment {
                TOKEN = credentials('github-token')
            }
            steps {
                sh """
                rm -rf manifests
                git clone --branch $MANIFEST_REPO_BRANCH $MANIFEST_REPO_URL
                cd manifests
                git config user.name "jenkins"
                git config user.email "jenkins@ci.local"

                git checkout -b update-image-$BUILD_NUMBER
                sed -i "s|image: .*|image: ${FULL_IMAGE}|" deployment.yaml

                git add deployment.yaml
                git commit -m "Update image to ${FULL_IMAGE}"
                git push https://$TOKEN@github.com/SUBASHREE-KB/manifests.git update-image-$BUILD_NUMBER
                """
            }
        }
    }
}

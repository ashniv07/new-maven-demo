pipeline {
    agent any

    environment {
        IMAGE_NAME = "backend-app" // Just image name, no URL here
        IMAGE_TAG = "v${BUILD_NUMBER}"
        FULL_IMAGE = "3.142.249.69:5000/${IMAGE_NAME}:${IMAGE_TAG}"

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

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}") // Build only, locally
                }
            }
        }

        stage('Push Docker Image to Nexus') {
            environment {
                DOCKER_CREDS = credentials('nexus-docker-credentials') // ID from Jenkins credentials
            }
            steps {
                script {
                    docker.withRegistry('http://3.142.249.69:5000', 'nexus-docker-credentials') {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
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
                sed -i "s|image: .*|image: 3.142.249.69:5000/${IMAGE_NAME}:${IMAGE_TAG}|" deployment.yaml

                git add deployment.yaml
                git commit -m "Update image to 3.142.249.69:5000/${IMAGE_NAME}:${IMAGE_TAG}"
                git push https://$TOKEN@github.com/SUBASHREE-KB/manifests.git update-image-$BUILD_NUMBER
                """
            }
        }
    }
}

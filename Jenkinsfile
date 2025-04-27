pipeline {
    agent any

    environment {
        IMAGE_NAME = "3.142.249.69:5000/backend-app"
        IMAGE_TAG = "v${BUILD_NUMBER}"
        FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
        APP_REPO_URL = "https://github.com/SUBASHREE-KB/applicationcode.git"
        MANIFEST_REPO_URL = "https://github.com/SUBASHREE-KB/manifests.git"
        MANIFEST_REPO_BRANCH = "main"
        DOCKER_CREDS = credentials('nexus-docker-credentials')
        TOKEN = credentials('github-token')
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                git url: "${APP_REPO_URL}", branch: 'main'
                sh 'ls -al'  // Verify Dockerfile is present
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Create a temporary script file
                    writeFile file: './docker_build_push.sh', text: '''#!/bin/bash
set -e

# Log Docker version and configuration
echo "Docker version:"
docker --version
echo "Docker configuration:"
cat /etc/docker/daemon.json 2>/dev/null || echo "No daemon.json found"

# Login to Docker registry
echo "Logging in to Docker registry..."
echo $1 | docker login $2 -u $3 --password-stdin

# Build the Docker image
echo "Building Docker image: $4"
docker build -t $4 .

# Push the Docker image
echo "Pushing Docker image to registry..."
docker push $4

echo "Docker build and push completed successfully"
'''
                    
                    // Make the script executable
                    sh 'chmod +x ./docker_build_push.sh'
                    
                    // Execute the script with parameters
                    sh './docker_build_push.sh "$DOCKER_CREDS_PSW" "3.142.249.69:5000" "$DOCKER_CREDS_USR" "$FULL_IMAGE"'
                }
            }
        }

        stage('Clone Manifest Repo and Update Image Tag') {
            steps {
                script {
                    // Create manifest update script
                    writeFile file: './update_manifest.sh', text: '''#!/bin/bash
set -e

# Variables from parameters
MANIFEST_REPO_URL=$1
MANIFEST_REPO_BRANCH=$2
BUILD_NUMBER=$3
FULL_IMAGE=$4
GITHUB_TOKEN=$5

echo "Cloning manifest repository..."
rm -rf manifests
git clone --branch ${MANIFEST_REPO_BRANCH} ${MANIFEST_REPO_URL}
cd manifests

echo "Configuring Git user..."
git config user.name "jenkins"
git config user.email "jenkins@ci.local"

echo "Creating new branch..."
git checkout -b update-image-${BUILD_NUMBER}

echo "Updating deployment.yaml..."
sed -i "s|image: .*|image: ${FULL_IMAGE}|" deployment.yaml

echo "Committing changes..."
git add deployment.yaml
git commit -m "Update image to ${FULL_IMAGE}"

echo "Pushing changes to GitHub..."
git push https://${GITHUB_TOKEN}@github.com/SUBASHREE-KB/manifests.git update-image-${BUILD_NUMBER}

echo "Manifest update completed successfully"
'''

                    // Make the script executable
                    sh 'chmod +x ./update_manifest.sh'
                    
                    // Execute the script with parameters
                    sh './update_manifest.sh "$MANIFEST_REPO_URL" "$MANIFEST_REPO_BRANCH" "$BUILD_NUMBER" "$FULL_IMAGE" "$TOKEN"'
                }
            }
        }
        
        stage('Diagnostics') {
            steps {
                sh '''
                    echo "Jenkins is running as: $(whoami)"
                    echo "Groups: $(groups)"
                    echo "Docker socket permissions: $(ls -la /var/run/docker.sock)"
                    echo "HOME directory: $HOME"
                    echo "Docker config: $(ls -la $HOME/.docker 2>/dev/null || echo 'No Docker config')"
                '''
            }
        }
    }
    
    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the logs for details."
        }
        always {
            // Clean up temporary scripts
            sh 'rm -f ./docker_build_push.sh ./update_manifest.sh'
        }
    }
}
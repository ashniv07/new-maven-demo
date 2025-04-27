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
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                git url: "${APP_REPO_URL}", branch: 'main'
                sh 'ls -al'  // Verify Dockerfile is present
            }
        }

       stage('Build and Push Docker Image') {
            environment {
                DOCKER_CREDS = credentials('nexus-docker-credentials')
                REGISTRY_URL = "http://3.142.249.69:5000"
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-docker-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh '''
                        echo "${DOCKER_PASSWORD}" | docker login ${REGISTRY_URL} -u "${DOCKER_USERNAME}" --password-stdin
                        docker build -t $FULL_IMAGE .
                        docker push $FULL_IMAGE
                    '''
                }
            }
        }
        stage('Clone Manifest Repo and Update Image Tag') {
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

pipeline {
    agent {
        label 'agent-node1'
    }
    
    environment {
        APP_REPO = 'https://github.com/sandyxd18/flask-todolist.git'
        MANIFEST_REPO = 'https://github.com/sandyxd18/manifest-tubes.git'
        DOCKER_IMAGE = 'sandyxd18/backend-tubes'
        IMAGE_TAG = ''
        GIT_CREDENTIALS_ID = credentials('github-creds')
        DOCKER_CREDENTIALS_ID = credentials('a77b13b7-6e11-4652-9760-e73ef64c6d32')
        ARGOCD_CREDENTIALS_ID = credentials('argocd-creds')
        ARGOCD_SERVER = '192.168.22.172:31014'
        APP_NAME = 'backend'
    }
    
    stages {
        stage('Clone Backend Repo') {
            steps {
                git branch: 'main', url: "${APP_REPO}"
            }
        }
        
        stage('Build and Push Docker Images') {
            steps {
                script {
                    IMAGE_TAG = "v${env.BUILD_NUMBER}"
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                            echo $DOCKER_CREDENTIALS_ID | docker login -u $DOCKER_CREDENTIALS_ID -p $DOCKER_CREDENTIALS_ID
                            docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }
        
        stage('Clone Manifest Repo') {
            steps {
                dir('manifest') {
                    git url: "${MANIFEST_REPO}", credentialsId: "${GIT_CREDENTIALS_ID}"
                }
            }
        }
        
        stage('Update Image Tag') {
            steps {
                dir('manifest/backend-manifest') {
                    script {
                        sh """
                            sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${IMAGE_TAG}|' backend-deployment.yaml
                            git config user.name "jenkins"
                            git config user.email "jenkins@example.com"
                            git add .
                            git commit -m "Update image tag to ${IMAGE_TAG}"
                            git push origin main
                        """
                    }
                }
            }
        }
        
        stage('Sync ArgoCD') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: '${ARGOCD_CREDENTIALS_ID}', usernameVariable: 'ARGOCD_USER', passwordVariable: 'ARGOCD_PASS')]) {
                        sh """
                            argocd login $ARGOCD_SERVER --username $ARGOCD_USER --password $ARGOCD_PASS --insecure
                            argocd app sync ${APP_NAME}
                        """
                    }
                }
            }
        }
    }
}
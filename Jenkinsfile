pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    tty: true
  - name: git
    image: alpine/git
    command:
    - cat
    tty: true
  - name: argocd
    image: argoproj/argocd:latest
    command:
    - sleep
    - infinity
    tty: true
"""
        }
    }

    environment {
        APP_REPO = 'https://github.com/sandyxd18/flask-todolist.git'
        MANIFEST_REPO = 'https://github.com/sandyxd18/manifest-tubes.git'
        DOCKER_IMAGE = 'sandyxd18/backend-tubes'
        IMAGE_TAG = ''
        GIT_CREDENTIALS_ID = 'github-creds'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
        ARGOCD_CREDENTIALS_ID = 'argocd-creds'
        ARGOCD_SERVER = '192.168.22.172:31014'
        APP_NAME = 'backend'
    }

    stages {
        stage('Clone Backend Repo') {
            steps {
                container('git') {
                    git branch: 'main', url: "${APP_REPO}", credentialsId: "${GIT_CREDENTIALS_ID}"
                }
            }
        }

    stage('Build and Push Docker Image') {
        steps {
            container('kaniko') {
                script {
                    IMAGE_TAG = "v${env.BUILD_NUMBER}"
                    sh """
                        /kaniko/executor \
                        --dockerfile=Dockerfile \
                        --context=dir://${env.WORKSPACE} \
                        --destination=${DOCKER_IMAGE}:${IMAGE_TAG} \
                        --skip-tls-verify
                    """
                }
            }
        }
    }

        stage('Clone Manifest Repo') {
            steps {
                container('git') {
                    dir('manifest') {
                        git url: "${MANIFEST_REPO}", credentialsId: "${GIT_CREDENTIALS_ID}"
                    }
                }
            }
        }

        stage('Update Image Tag') {
            steps {
                container('git') {
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
        }

        stage('Sync ArgoCD') {
            steps {
                container('argocd') {
                    withCredentials([usernamePassword(credentialsId: "${ARGOCD_CREDENTIALS_ID}", usernameVariable: 'ARGOCD_USER', passwordVariable: 'ARGOCD_PASS')]) {
                        sh """
                          argocd login ${ARGOCD_SERVER} --username $ARGOCD_USER --password $ARGOCD_PASS --insecure
                          argocd app sync ${APP_NAME}
                        """
                    }
                }
            }
        }
    }
}

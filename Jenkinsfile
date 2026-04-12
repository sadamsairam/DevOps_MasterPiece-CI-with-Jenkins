peline {
    agent any

    environment {
        NAME = "spring-app"
        VERSION = "${env.BUILD_ID}"
        IMAGE_REPO = "praveensirvi"
        GIT_REPO_NAME = "DevOps_MasterPiece-CD-with-argocd"
        GIT_USER_NAME = "praveensirvi1212"
    }

    tools {
        maven 'maven-3.8.6'
    }

    stages {

        stage('Checkout git') {
            steps {
                git branch: 'main', url:'https://github.com/praveensirvi1212/DevOps_MasterPiece-CI-with-Jenkins.git'
            }
        }

        stage('Set Commit ID') {
            steps {
                script {
                    env.GIT_COMMIT = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                }
            }
        }

        stage ('Build & JUnit Test') {
            steps {
                sh 'mvn clean install'
            }
        }


        stage('Docker Build') {
            steps {
                sh """
                docker build -t ${IMAGE_REPO}/${NAME}:${VERSION}-${GIT_COMMIT} .
                """
            }
        }

        stage ('Docker Image Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'sairamsadamss', usernameVariable: 'DOCKER_USER', passwordVariable: 'Sairam@12345')]) {
                    sh """
                    docker login -u $DOCKER_USER -p $DOCKER_PASS
                    docker push ${IMAGE_REPO}/${NAME}:${VERSION}-${GIT_COMMIT}
                    docker rmi ${IMAGE_REPO}/${NAME}:${VERSION}-${GIT_COMMIT}
                    """
                }
            }
        }

        stage('Clone/Pull k8s deployment Repo') {
            steps {
                script {
                    if (fileExists('DevOps_MasterPiece-CD-with-argocd')) {
                        dir("DevOps_MasterPiece-CD-with-argocd") {
                            sh 'git pull'
                        }
                    } else {
                        sh 'git clone -b feature https://github.com/praveensirvi1212/DevOps_MasterPiece-CD-with-argocd.git'
                    }
                }
            }
        }

        stage('Update deployment Manifest') {
            steps {
                dir("DevOps_MasterPiece-CD-with-argocd/yamls") {
                    sh """
                    sed -i "s#${IMAGE_REPO}/${NAME}:.*#${IMAGE_REPO}/${NAME}:${VERSION}-${GIT_COMMIT}#g" deployment.yaml
                    cat deployment.yaml
                    """
                }
            }
        }

        stage('Commit & Push changes') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    dir("DevOps_MasterPiece-CD-with-argocd") {
                        sh """
                        git config user.email "praveen@gmail.com"
                        git config user.name "praveen"

                        git checkout feature
                        git add .
                        git commit -m "Updated image ${VERSION}-${GIT_COMMIT}" || true
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} feature
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline Completed"
        }
    }
}

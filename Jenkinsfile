pipeline {
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

        stage('Clean Workspace (FULL RESET)') {
            steps {
                cleanWs()
                deleteDir()
            }
        }

        stage('Checkout Latest Code (FORCE FRESH)') {
            steps {
                git branch: 'main',
                url: 'https://github.com/sadamsairam/DevOps_MasterPiece-CI-with-Jenkins.git'
            }
        }

        stage('Force Git Latest Commit') {
            steps {
                script {
                    sh 'git fetch --all'
                    sh 'git reset --hard origin/main'
                    env.GIT_COMMIT = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                    echo "Using commit: ${env.GIT_COMMIT}"
                }
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                docker build --no-cache \
                -t ${IMAGE_REPO}/${NAME}:${VERSION}-${GIT_COMMIT} .
                """
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'sairamsadamss',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${IMAGE_REPO}/${NAME}:${VERSION}-${GIT_COMMIT}
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline Completed"
            cleanWs()
        }
    }
}


pipeline {
    agent any

    tools {
        jdk 'JDK'
        maven 'MAVEN_HOME'
    }

    environment {
        MAVEN_OPTS = '-Xmx1024m'
        BACKEND_IMAGE = 'yasmine12346/backend-app'
        FRONTEND_IMAGE = 'yasmine12346/frontend-app'
        COMPOSE_PROJECT_NAME = 'devops-appgestiondesprojets'
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Checkout GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/yasminebenattia/DevOps-AppGestionDesProjets.git'
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Tests unitaires') {
            steps {
                dir('backend') {
                    sh 'mvn test'
                }
            }
            post {
                always {
                    junit 'backend/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package Backend') {
            steps {
                dir('backend') {
                    sh 'mvn package -DskipTests'
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: 'backend/target/*.jar',
                        fingerprint: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    env.IMAGE_TAG = env.BUILD_NUMBER
                }

                sh '''
                    docker build \
                        -t $BACKEND_IMAGE:$IMAGE_TAG \
                        backend

                    docker build \
                        -t $FRONTEND_IMAGE:$IMAGE_TAG \
                        frontend
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                        set -e

                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin

                        trap 'docker logout >/dev/null 2>&1 || true' EXIT

                        docker push $BACKEND_IMAGE:$IMAGE_TAG
                        docker push $FRONTEND_IMAGE:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh '''
                    export IMAGE_TAG=$BUILD_NUMBER

                    docker compose \
                        -p $COMPOSE_PROJECT_NAME \
                        up -d --force-recreate
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline réussie : Backend et Frontend construits, publiés sur Docker Hub et déployés avec Docker Compose.'
        }

        failure {
            echo 'La pipeline a échoué. Vérifier les logs des étapes.'
        }
    }
}

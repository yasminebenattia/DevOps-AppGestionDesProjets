
pipeline {
    agent any

    tools {
        jdk 'JDK'
        maven 'MAVEN_HOME'
    }

    environment {
        MAVEN_OPTS = '-Xmx1024m'
        DOCKER_IMAGE = 'yasmine12346/backend-app'
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

        stage('Package') {
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
                    env.IMAGE_TAG = "${env.BUILD_NUMBER}"
                }
                sh '''
                    docker build -t $DOCKER_IMAGE:$IMAGE_TAG backend
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
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE:$IMAGE_TAG
                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline réussie : backend compilé et image publiée sur Docker Hub.'
        }
        failure {
            echo 'La pipeline a échoué.'
        }
    }
}


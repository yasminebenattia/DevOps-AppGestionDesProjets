
pipeline {
    agent any

    tools {
        jdk 'JDK'
        maven 'MAVEN_HOME'
    }

    environment {
        MAVEN_OPTS = '-Xmx1024m'
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
        }

        stage('Package') {
            steps {
                dir('backend') {
                    sh 'mvn package -DskipTests'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline terminee avec succes !'
        }
        failure {
            echo 'La pipeline a echoue.'
        }
    }
}

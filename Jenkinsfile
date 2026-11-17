pipeline {
    agent any

    tools {
        jdk 'JDK'
        maven 'MAVEN_HOME'
    }

    environment {
        MAVEN_OPTS = '-Xmx1024m'
    }

    triggers {
        githubPush()              // déclenché par le webhook GitHub
        // pollSCM('H/2 * * * *') // solution de secours si le webhook n'est pas joignable
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
                    archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true
                }
            }
        }
    }

    post {
        success { echo 'Pipeline réussie : le livrable est dans backend/target.' }
        failure { echo 'La pipeline a échoué.' }
    }
}

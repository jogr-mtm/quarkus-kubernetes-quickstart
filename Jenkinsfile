pipeline {
    agent any
    triggers {
        cron('H H(21-23) */2 * *')
    }
    tools {
        jdk 'graalvm'
    }
    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '15'))
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jogr-mtm/quarkus-kubernetes-quickstart.git'
            }
        }
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    which java
                '''
            }
        }
        stage('quarkus:dev') {
            options {
                timeout(time: 30, unit: 'MINUTES')
            }
            steps {
                sh './mvnw -fn clean package -Pnative'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
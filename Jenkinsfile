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
        stage('Test Native') {
            options {
                timeout(time: 3, unit: 'HOURS')
            }
            steps {
                sh 'GRAALVM_HOME=$JAVA_HOME TESTCONTAINERS_RYUK_DISABLED=true ./mvnw -fn clean verify -Dnative'
            }
            post {
                always {
                    junit '**/target/*-reports/TEST*.xml'
                    sh '''
                        for i in `find . | grep 'runner$' | grep -v "wiring-classes" | grep -v "test-classes" | grep -v "generated-sources" | sort`; do
                            QS_DIR="`dirname $i`"
                            NATIVE_SIZE=`du -m $i | cut -f1`
                            echo "$QS_DIR,$NATIVE_SIZE Mbyte"
                        done > disk-usage-native-tmp.txt

                        column -t -s',' disk-usage-native-tmp.txt > disk-usage-native.txt
                        rm disk-usage-native-tmp.txt
                    '''
                }
            }
        }
        stage('Reports') {
            parallel {
                stage('Disk usage') {
                    steps {
                        sh 'du -cskh */*'
                    }
                }
                stage('Archive artifacts') {
                    steps {
                        archiveArtifacts artifacts: '**/target/*-reports/TEST*.xml,disk-usage-jvm.txt,disk-usage-native.txt', fingerprint: false
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
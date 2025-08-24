pipeline {
    agent any

    environment {
        JAR_NAME = "target/demo-0.0.1-SNAPSHOT.jar"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Shubhan-siri/ci-cd-new-boot-app.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean verify'
            }
        }

        stage('Package') {
            steps {
                sh """
                    cp $JAR_NAME app.jar
                """
                archiveArtifacts artifacts: 'app.jar', fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                // Stop existing app if running
                sh 'pkill -f "java -jar app.jar" || true'

                // Start app in background on port 9090
                sh 'nohup java -jar app.jar --server.port=9090 > app.log 2>&1 &'
            }
        }

        stage('Smoke Test') {
            steps {
                sh 'curl -sSf http://localhost:9090/ | head -n 1'
            }
        }
    }

    post {
        always {
            echo 'Last 20 lines of app.log (if present):'
            sh 'tail -n 20 app.log || true'
        }
    }
}

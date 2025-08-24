pipeline {
  agent any
  options { timestamps(); disableConcurrentBuilds() }
  tools { maven 'M3' }
  environment { APP_PORT = '9090' }
  triggers { pollSCM('H/2 * * * *') }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Build & Test') { steps { sh 'mvn -B clean verify' } }
    stage('Package & Archive') {
      steps {
        sh '''
          mkdir -p target
          JAR=$(ls target/*SNAPSHOT.jar 2>/dev/null | head -n 1)
          if [ -z "$JAR" ]; then JAR=$(ls target/*.jar | head -n 1); fi
          cp "$JAR" app.jar
        '''
        archiveArtifacts artifacts: 'app.jar', fingerprint: true
      }
    }
    stage('Deploy') {
      steps {
        sh '''
          pkill -f "java -jar app.jar" || true
          nohup java -jar app.jar --server.port=${APP_PORT} > app.log 2>&1 &
          sleep 3
          pgrep -f "java -jar app.jar" >/dev/null && echo "App started" || (echo "App failed"; tail -n 100 app.log; exit 1)
        '''
      }
    }
    stage('Smoke Test') {
      steps {
        sh 'curl -sSf http://localhost:${APP_PORT}/ | head -n 1'
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

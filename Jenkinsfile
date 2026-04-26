pipeline{
  agent any 
  tools{
    jdk 'JDK17'
    maven 'MVN3'
  }
  stages{
    stage('Clean Workspace'){
      steps{
        cleanWs()
      }
    }
    stage('Checkout SCM'){
      steps{
        git branch: 'java', url: 'https://github.com/tawfeeq421/e-com-app.git'
      }
    }
    stage('Build'){
      steps{
        sh 'mvn clean install -DskipTests'
      }
    }
    stage('Unit Tests'){
      steps{
        sh 'mvn test'
      }
    }
    stage('SonarQube Analysis'){
      environment{
        scannerHome = tool 'sonar'
      }
      steps{
        withSonarQubeEnv('sonarserver'){
          sh """
          ${scannerHome}/bin/sonar-scanner \
          -Dsonar.projectKey=ecom-app2 \
          -Dsonar.projectName=ecom-app2 \
          -Dsonar.sources=. \
          -Dsonar.java.binaries=target/classes
          """
        }
      }
    }
    stage('Quality Gate'){
      steps{
        timeout(time: 1, unit: 'HOURS'){
          waitForQualityGate abortPipeline: true
        }
      }
    }
    stage('Trivy FS Scan'){
      steps{
        sh '''
        trivy fs . \
        --severity HIGH,CRITICAL \
        --format table \
        --no-progress \
        -o trivy-report.txt 
        '''
      }
    }
  }
  post {
    always{
      archiveArtifacts artifacts: 'trivy-report.txt', fingerprint: true
    }
    success{
      slackSend(
        channel: "#e-com",
        color: "good",
        message: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER} ${env.BUILD_URL}"
      )
    }
    failure{
      slackSend(
        channel: "#e-com",
        color: "danger",
        message: "❌ FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}\nCheck Logs ${env.BUILD_URL}"
      )
    }
  }
}
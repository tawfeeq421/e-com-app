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
        sh 'mvn clean install -DskipTests=true'
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
          ${scannerHome}/bin/sonar-scanner
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
        --format html \
        --no-progress \
        -o trivy-report.html 
        '''
      }
    }
  }
}
pipeline {
  agent any
  tools{
    jdk 'JDK17'
    nodejs 'NODE18'
  }
  stages{
    stage('Clean Workspace'){
      steps{
        cleanWs()
      }
    }
    stage('Checkout SCM'){
      steps{
        git branch: 'main', url: 'https://github.com/tawfeeq421/e-com-app.git'
      }
    }
    stage('Install Dependency'){
      steps{
        dir('client'){
          sh '''
          npm install 
          npm install @angular/cli
          '''
        }
      }
    }
    stage('Buld Artifacts'){
      steps{
        dir('client'){
          sh '''
          npm install
          npx ng build --configuration production
          '''
        }
      }
    }
    stage('Run Tests'){
      steps{
        dir('client'){
          sh 'ng test --watch=false --browsers=ChromeHeadless || true'
        }
      }
    }
    stage('SonarQube Analysis'){
      environment{
        scannerHome = tool 'sonar'
      }
      steps{
        dir('client'){
          withSonarQubeEnv('sonarserver'){
            sh "${scannerHome}/bin/sonar-scanner"
          }
        }
      }
    }
    stage('Quality Gate'){
      steps{
        dir('client'){
          timeout(time: 1, unit: 'HOURS'){
            waitForQualityGate abortPipeline: true
          }
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
  post{
    always{
      archiveArtifacts artifacts: 'trivy-report.txt' fingerprint: true
    }
    success{
      slackSend(
        channel: "#e-com"\
        color: "good"
        message: "✅ SUCCESS: ${env.JOB_NAME} ${env.BUILD_NUMBER}\n url ${env.BUILD_URL}"
      )
    }
    failure{
      slackSend(
        channel: "#e-com"
        color: "danger"
        message: "❌ FAILURE: ${env.JOB_NAME} ${env.BUILD_NUMBER}\nCheck Logs ${env.BUILD_URL}"
      )
    }
  }
}
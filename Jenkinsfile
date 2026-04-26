pipeline{
    agent any
    tools {
        jdk 'JDK17'
        nodejs 'NODE16'
    }
    stages{
        stage('Clean Workspace'){
            steps{
                cleanWs
            }
        }
        stage('Checkout SCM'){
            steps{
                git branch: 'node', url: 'https://github.com/tawfeeq421/e-com-app.git'
            }
        }
        stage('Install Dependency'){
            steps{
                sh 'npm install'
            }
        }
        stage('SonarQube Analysis'){
            environment{
                scannerHome = tool 'sonar'
            }
            steps{
                withSonarQubeEnv('sonarserver'){
                    sh "${scannerHome}/bin/sonar-scanner" 
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
    post{
        always{
            archiveArtifacts artifacts: 'trivy-report.html', fingerprint: true
        }
        success{
            slackSend(
                channel: "#e-com"
                color: "good"
                message: "✅ SUCCESS: ${env.JOB_NAME} ${env.BUILD_NUMBER}\n See Lgs ${env.BUILD_URL}"
            )
            failure{
                slackSend(
                    channel: "#e-com"
                    color: "danger"
                    message: "❌ FAILURE: ${env.JOB_NAME} ${env.BUILD_NUMBER}\n see logs ${env.BUILD_URL}"
                )
            }
        }
    }
}
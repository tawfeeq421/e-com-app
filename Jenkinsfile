pipeline {
  agent any
  tools{
    jdk 'JDK17'
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
    stage('')
  }
}
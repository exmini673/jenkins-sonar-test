node {
  stage('SCM') {
    checkout scm
  }
  stage('SonarQube Analysis') {
      steps {
          steps {
              script {
                  withSonarQubeEnv('sonar-pr') {
                      sh 'mvn sonar:sonar -Dsonar.projectKey=pr-project 
                  }
              }
              timeout(time:1, unit: 'MINUTES') {
                  waitForQualityGate abortPipeline: true 
              }
          }
      }

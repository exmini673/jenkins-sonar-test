pipeline {
    agent any // Specify the agent type you want to use

    environment {
        GITHUB_CREDENTIAL = credentials('jenkins-sonar-token')
    }

    triggers {
        githubPush()
    }

    options {
        timestamps()
    }

    stages {
        stage('Compile') {
            steps {
                sh "chmod +x mvnw"
                sh "./mvnw clean"
                sh "./mvnw compile"
            }
        }
        stage('Testing & QC') {
            steps {
                script {
                    withSonarQubeEnv {
                        sh "./mvnw verify sonar:sonar -Dsonar.java.source=17 \
                            -Dsonar.projectKey=sonar_project01 -Dsonar.sources=src/main/ -Dsonar.tests=src/test/ \
                            -Dsonar.java.binaries=target"
                    }
                }
            }
        }
    }
}


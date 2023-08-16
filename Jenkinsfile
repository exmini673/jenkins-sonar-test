pipeline {
    agent {
        docker {
            image 'maven:3.8.3-openjdk-17'
            reuseNode true
            registryUrl 'https://index.docker.io/v1/'
            registryCredentialsId 'docker-hub'
        }
    }
    environment {
        GC = credentials('jenkins-sonar-token') // 생성
        GIT_REPO = 'jenkins-sonar-test'
        GIT_USERNAME = 'exmini673'
        TAG_VERSION = 'v1.0.0'
    }

    triggers {
        githubPush()
    }
    stages {
        stage('maven build, test, packageing(war)'){
            steps {
            sh 'mvn clean install'
           }
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


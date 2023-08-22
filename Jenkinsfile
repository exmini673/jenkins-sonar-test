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
    options {
        // 트리거 발생할 때 동작하는 기본 체크아웃 과정 생략
        skipDefaultCheckout(true)
    }
    stages {
        // 기본 체크아웃 대신 동작할 스테이지
        stage("GitHub dev branch checkout") {
            steps {
                checkout scm: scmGit(
                    userRemoteConfigs: [
                        [
                            credentialsId: "jenkins-sonar-token",
                            url: "https://github.com/exmini673/${GIT_REPO}.git"
                        ]
                    ],
                    branches: [
                        [
                            name: "dev"
                        ]
                    ]
                )
            }
        }  
        stage('maven build, test, packageing(war)') {
            steps {
                sh 'mvn clean install'
            }
        }
        
        stage('Testing & QC') {
            steps {
                script {
                    withSonarQubeEnv('sonar-pr') {
                       sh """
                            docker run --rm \
                              -e SONAR_HOST_URL=$SONAR_HOST_URL \
                              -e SONAR_LOGIN=$SONAR_AUTH_TOKEN \
                              -e SONAR_SCANNER_OPTS='-Dsonar.projectKey=pr-project  -Dsonar.verbose=true' \
                              -v \$(pwd):/usr/src \
                              sonarsource/sonar-scanner-cli
                        """
                    }
                }
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('PR Decoration') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                         sh """
                            docker run --rm \
                              -e SONAR_HOST_URL=$SONAR_HOST_URL \
                              -e SONAR_LOGIN=$SONAR_AUTH_TOKEN \
                              -e SONAR_SCANNER_OPTS='-Dsonar.projectKey=sonar_project01  -Dsonar.verbose=true' \
                              -v \$(pwd):/usr/src \
                              sonarsource/sonar-scanner-cli
                        """
                    }
                }
                timeout(time: 1, unit: 'MINUTES') {
                    // waitForQualityGate abortPipeline: true
                }
            }
        }
        // stage('github create release') {
        //     steps {
        //         script {
        //              def response = sh(script: """
        //                 curl -sSL \
        //                     -X POST \
        //                     -H "Accept: application/vnd.github+json" \
        //                     -H "Authorization: Bearer ${GC_PSW}" \
        //                     -H "X-GitHub-Api-Version: 2022-11-28" \
        //                     https://api.github.com/repos/${GIT_USERNAME}/${GIT_REPO}/releases \
        //                     -d '{
        //                             "tag_name":"${TAG_VERSION}",
        //                             "target_commitish":"main",
        //                             "name":"Release ${TAG_VERSION}",
        //                             "body":"Description of the release",
        //                             "draft":false,
        //                             "prerelease":false,
        //                             "generate_release_notes":false
        //                         }'
        //             """, returnStdout: true)

        //             def json = readJSON text: "$response"
        //             def id = json.id

        //             sh "mv target/demo-0.0.1-SNAPSHOT.war ${GIT_REPO}-${TAG_VERSION}.war"

        //             sh """
        //                 curl -sSL \
        //                     -X POST \
        //                     -H "Accept: application/vnd.github+json" \
        //                     -H "Authorization: Bearer ${GC_PSW}" \
        //                     -H "X-GitHub-Api-Version: 2022-11-28" \
        //                     -H "Content-Type: application/octet-stream" \
        //                     "https://uploads.github.com/repos/${GIT_USERNAME}/${GIT_REPO}/releases/${id}/assets?name=${GIT_REPO}-${TAG_VERSION}.war" \
        //                     --data-binary "@${GIT_REPO}-${TAG_VERSION}.war"
        //             """
                    
        //         }
        //     }       
        // }
    }
}

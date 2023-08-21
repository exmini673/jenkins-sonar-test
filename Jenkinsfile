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
        stage('maven build, test, packageing(war)') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Testing & QC') {
           steps {
              script {
                  withSonarQubeEnv('sonar') {
                      sh """
                          docker run --rm \
                            -e SONAR_HOST_URL=$SONAR_HOST_URL \
                            -e SONAR_LOGIN=$SONAR_AUTH_TOKEN \
                            -e SONAR_SCANNER_OPTS='-Dsonar.verbose=true -Dsonar.projectKey=sonar_project01' \
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
        // stage('압축한 소스 코드 도커 이미지로 빌드 및 푸쉬') {
        //     steps {
        //         sh "docker login -u ${DOCKER_CREDENTIAL_USR} -p ${DOCKER_CREDENTIAL_PSW}"
        //         sh "docker build -t ${DOCKER_CREDENTIAL_USR}/${PROJECT_NAME}:latest -f .docker/Dockerfile ."
        //         sh "docker tag ${DOCKER_CREDENTIAL_USR}/${PROJECT_NAME}:latest ${DOCKER_CREDENTIAL_USR}/${PROJECT_NAME}:${TAG_VERSION}"
        //         sh "docker push ${DOCKER_CREDENTIAL_USR}/${PROJECT_NAME}:${TAG_VERSION}"
        //         sh "docker push ${DOCKER_CREDENTIAL_USR}/${PROJECT_NAME}:latest"
        //     }
        // }
        // stage('도커 허브에 푸쉬한 이미지로 docker-server 에 동작') {
        //     steps {
        //         sh "docker -H tcp://docker-server:2375 stop web1"
        //         sh "docker -H tcp://docker-server:2375 rm web1"
        //         sh """
        //             docker -H tcp://docker-server:2375 run -it -d -p 8080:80 --name web1 \
        //             hiwill41/static-web:${TAG_VERSION}
        //         """
        //     }
        // }
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

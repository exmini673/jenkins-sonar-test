pipeline {
    agent {
        docker {
            image 'maven:3.8.3-openjdk-17'
            reuseNode true
            registryUrl 'https://index.docker.io/v1/'
            registryCredentialsId 'docker-hub'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
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
        stage('maven build, test, packaging(war)') {
            steps {
                sh 'mvn clean install'
            }
        }
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
        stage('github create release') {
            steps {
                script {
                     def response = sh(script: """
                        curl -sSL \
                            -X POST \
                            -H "Accept: application/vnd.github+json" \
                            -H "Authorization: Bearer ${GC_PSW}" \
                            -H "X-GitHub-Api-Version: 2022-11-28" \
                            https://api.github.com/repos/${GIT_USERNAME}/${GIT_REPO}/releases \
                            -d '{
                                    "tag_name":"${TAG_VERSION}",
                                    "target_commitish":"main",
                                    "name":"Release ${TAG_VERSION}",
                                    "body":"Description of the release",
                                    "draft":false,
                                    "prerelease":false,
                                    "generate_release_notes":false
                                }'
                    """, returnStdout: true)

                    def json = readJSON text: "$response"
                    def id = json.id

                    sh "mv target/demo-0.0.1-SNAPSHOT.war ${GIT_REPO}-${TAG_VERSION}.war"

                    sh """
                        curl -sSL \
                            -X POST \
                            -H "Accept: application/vnd.github+json" \
                            -H "Authorization: Bearer ${GC_PSW}" \
                            -H "X-GitHub-Api-Version: 2022-11-28" \
                            -H "Content-Type: application/octet-stream" \
                            "https://uploads.github.com/repos/${GIT_USERNAME}/${GIT_REPO}/releases/${id}/assets?name=${GIT_REPO}-${TAG_VERSION}.war" \
                            --data-binary "@${GIT_REPO}-${TAG_VERSION}.war"
                    """
                    
                }
            }       
        }
    }
}

pipeline {
    agent any
    environment {
        PROJECT_BRANCH_NAME = "${env.BRANCH_NAME.replaceAll('/', '-')}"
        PROJECT_BUILD = "${env.PROJECT_BRANCH_NAME}.${env.BUILD_NUMBER}"
        DOCKER_IMAGE_NAME="vema-zuers-service"
    }
    stages {
        stage('Build') {
            agent {
                dockerfile {
                    filename 'Dockerfile'
                    dir 'docker'
                    args '-v /root/.m2/:/root/.m2 -v "/opt/docker/jenkins/home${WORKSPACE//${JENKINS_HOME}/}":/usr/src/mymaven -w /usr/src/mymaven'
                }
            }
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    VERSION_POM = pom.version.replaceAll('-SNAPSHOT', '')
                    VERSION_BUILD = pom.version.replaceAll('SNAPSHOT', PROJECT_BUILD)
                }

                sh '''#!/bin/bash
                    echo "Buildumgebung"
                    echo "- PROJECT_BRANCH_NAME = ${PROJECT_BRANCH_NAME}"
                    echo "- PROJECT_BUILD = ${PROJECT_BUILD}"
                    echo
                '''

                sh "mvn -B org.codehaus.mojo:versions-maven-plugin:2.5:set -DnewVersion=${VERSION_BUILD}"

                configFileProvider(
                    [configFile(fileId: '67c71ee0-1e22-4627-afe1-2b0146c2afed', variable: 'MAVEN_SETTINGS')]) {
                    sh 'apt update && apt install -y graphviz && mvn -s $MAVEN_SETTINGS -B -DskipTests clean package'
                }
            }
        }
        stage('Test') {
            agent {
                dockerfile {
                    filename 'Dockerfile'
                    dir 'docker'
                    args '-v /root/.m2/:/root/.m2 -v "/opt/docker/jenkins/home${WORKSPACE//${JENKINS_HOME}/}":/usr/src/mymaven -w /usr/src/mymaven'
                }
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('JavaDocs') {
            agent {
                dockerfile {
                    filename 'Dockerfile'
                    dir 'docker'
                    args '-v /root/.m2/:/root/.m2 -v "/opt/docker/jenkins/home${WORKSPACE//${JENKINS_HOME}/}":/usr/src/mymaven -w /usr/src/mymaven'
                }
            }
            steps {
                script {
                    configFileProvider(
                                        [configFile(fileId: '67c71ee0-1e22-4627-afe1-2b0146c2afed', variable: 'MAVEN_SETTINGS')]) {
                                        sh 'apt update && apt install -y graphviz && mvn javadoc:javadoc'
                    }
                }
            }
            post {
                  always{
                         publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'javadoc/apidocs', reportFiles: 'index.html', reportName: 'javadoc', reportTitles: ''])
                        }
                  }
        }
        stage('Ascii-Docs'){
            agent {
                dockerfile {
                    filename 'Dockerfile'
                    dir 'docker'
                    args '-v /root/.m2/:/root/.m2 -v "/opt/docker/jenkins/home${WORKSPACE//${JENKINS_HOME}/}":/usr/src/mymaven -w /usr/src/mymaven'
                }
            }
            steps {
                 script {
                    configFileProvider(
                            [configFile(fileId: '67c71ee0-1e22-4627-afe1-2b0146c2afed', variable: 'MAVEN_SETTINGS')]) {
                            sh 'mvn asciidoctor:process-asciidoc'
                    }
                }
            }
          post {
          always{
                 publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target/generated-docs', reportFiles: 'Main.html', reportName: 'ASCIIdoc', reportTitles: ''])
                }
              }
        }


        stage('Deploy Docker-Image') {
            when {tag "v*"}
            environment {
                TAG = "${env.TAG_NAME.replaceFirst('v', '')}"
                USER_CI = credentials('174feafc-90f5-405b-838e-860b2ae12d02')
            }
              steps {
                    sh '''#!/bin/bash
                        # TODO: Hier machen wir das eig. doppelt, der Build liegt aber sonst nur im Container..
                        docker run -i  -v /root/.m2:/root/.m2 -v "/opt/docker/jenkins/home${WORKSPACE//${JENKINS_HOME}/}":/usr/src/mymaven -w /usr/src/mymaven maven:3.6-openjdk-15-slim mvn clean install -DskipTests
                        docker build --tag ${DOCKER_IMAGE_NAME}:topublish .

                        echo "Tagging images.."
                        docker tag ${DOCKER_IMAGE_NAME}:topublish docker.vemaeg.de/${DOCKER_IMAGE_NAME}:${VERSION_POM}
                        docker tag ${DOCKER_IMAGE_NAME}:topublish docker.vemaeg.de/${DOCKER_IMAGE_NAME}:latest

                        echo "Pushing images to nexus.."
                        docker login --username "${USER_CI_USR}" --password "${USER_CI_PSW}" docker.vemaeg.de
                        docker push docker.vemaeg.de/${DOCKER_IMAGE_NAME}:latest
                        docker push docker.vemaeg.de/${DOCKER_IMAGE_NAME}:${VERSION_POM}
                        docker logout docker.vemaeg.de

                        echo "Removing Images"
                        docker rmi docker.vemaeg.de/${DOCKER_IMAGE_NAME}:latest
                        docker rmi docker.vemaeg.de/${DOCKER_IMAGE_NAME}:${VERSION_POM}
                        docker rmi ${DOCKER_IMAGE_NAME}:topublish
                    '''
                        }

        }
    }
    post {
        always {
            cleanWs disableDeferredWipeout: true
        }
    }
    options {
        buildDiscarder logRotator(daysToKeepStr: '30', numToKeepStr: '10')
    }
    }

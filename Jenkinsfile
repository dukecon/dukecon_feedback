#!/usr/bin/env groovy

pipeline {
    agent {
        node {
            label 'docker'
        }
    }

    triggers {
        pollSCM('* * * * *')
    }


    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10', daysToKeepStr: '10'))
    }

    stages {
        stage('Build') {
            steps {
                withMaven {
                    script {
                        if (env.BRANCH_NAME == "develop") {
                            sh 'mvn -Ptestcontainers clean deploy'
                        } else {
                            sh 'mvn -Ptestcontainers clean verify'
                        }
                    }
                }
            }
        }
        stage('Promote Docker and restart latest') {
            when {
                branch 'develop'
            }
            steps {
                withMaven {
                    sh "mvn -Pdocker docker:build docker:push"
                    build 'docker_restart_develop_latest'
                }
            }
        }
        stage('Publish Docs') {
            steps {
                publishHTML(target: [allowMissing         : true,
                                     alwaysLinkToLastBuild: false,
                                     keepAll              : true,
                                     reportDir            : 'target/generated-docs/asciidoc',
                                     reportFiles          : 'api-guide.html',
                                     reportName           : 'REST API'
                   ]
                )
            }
        }
    }
    post {
        failure {
            // notify users when the Pipeline fails
            mail to: 'gerd@aschemann.net',
                    subject: "Failed DukeCon Feedback Pipeline: ${currentBuild.fullDisplayName}",
                    body: "Something is wrong with ${env.BUILD_URL}"
        }
    }
}

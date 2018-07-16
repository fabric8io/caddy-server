#!/usr/bin/groovy
@Library('github.com/fabric8io/fabric8-pipeline-library@master')
def utils = new io.fabric8.Utils()
def flow = new io.fabric8.Fabric8Commands()
def repo = 'caddy-server'
dockerTemplate{
    clientsNode{
        checkout scm
        if (utils.isCI()){
            echo 'No CI tests to run'

        } else if (utils.isCD()){
            echo 'Running CD pipeline'
            sh "git remote set-url origin git@github.com:fabric8io/${repo}.git"
            def newVersion = getNewVersion {}

            stage('tag') {
                container('clients') {
                    flow.setupGitSSH()
                    flow.pushTag(newVersion)
                }
            }

            stage('build'){
                container('docker') {
                     sh "docker build -t fabric8/${repo} ."
                }
            }

            stage('push to dockerhub'){
                container('docker') {
                    sh "docker push fabric8/${repo}:latest"
                    sh "docker tag fabric8/${repo}:latest fabric8/${repo}:${newVersion}"
                    sh "docker push fabric8/${repo}:${newVersion}"
                }
            }

            pushPomPropertyChangePR {
                propertyName = "${repo}.version"
                projects = [
                        'fabric8io/fabric8-devops'
                ]
                version = newVersion
            }
        }
    }
}

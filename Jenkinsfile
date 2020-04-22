#!/usr/bin/env groovy

pipeline {
    agent { label 'gelman-group-linux' }
    options {
        skipDefaultCheckout()
        preserveStashes(buildCount: 5)
    }
    parameters {
        string(defaultValue: '', name: 'major_version', description: "Major version of the docs to be built")
        string(defaultValue: '', name: 'minor_version', description: "Minor version of the docs to be built")
        string(defaultValue: '', name: 'last_docs_version_dir', description: "Last docs version found in /docs. Example: 2_21")
    }
    environment {
        GITHUB_TOKEN = credentials('6e7c1e8f-ca2c-4b11-a70e-d934d3f6b681')
    }
    stages {
        stage('Clean checkout for docs') {
            steps {
                deleteDir()
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/master']],
                          doGenerateSubmoduleConfigurations: false,
                          extensions: [],
                          submoduleCfg: [],
                          userRemoteConfigs: [[url: "https://github.com/stan-dev/docs.git", credentialsId: 'a630aebc-6861-4e69-b497-fd7f496ec46b']]]
                )
            }
        }
        stage("Create branch for docs") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'a630aebc-6861-4e69-b497-fd7f496ec46b',
                    usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh """#!/bin/bash
                        git checkout -b docs-$major_version-$minor_version
                    """
                }
            }
        }
        stage("Build docs") {
            steps{
                sh "python build.py $major_version $minor_version"
            }
        } 
        stage("Add redirects for docs") {
            steps{
                sh "python add_redirects.py $major_version $minor_version functions-reference"
                sh "python add_redirects.py $major_version $minor_version reference-manual"
                sh "python add_redirects.py $major_version $minor_version stan-users-guide"
            }
        }
        stage("Link docs to latest") {
            steps{
                sh "ls -lhart docs"
                sh "chmod +x add_links.sh"
                sh "./add_links.sh $last_docs_version_dir"
            }
        }
        stage("Push PR for docs") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'a630aebc-6861-4e69-b497-fd7f496ec46b',
                    usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh """#!/bin/bash
                        git config --global user.email "mc.stanislaw@gmail.com"
                        git config --global user.name "Stan Jenkins"
                        git config --global auth.token ${GITHUB_TOKEN}

                        git add .
                        git commit -m "Documentation generated from Jenkins for docs-$major_version-$minor_version"
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/stan-dev/docs.git docs-$major_version-$minor_version

                        curl -s -H "Authorization: token ${GITHUB_TOKEN}" -X POST -d '{"title": "Docs generated by Jenkins for v$major_version-$minor_version", "head":"docs-$major_version-$minor_version", "base":"master", "body":"Docs generated through the [Jenkins Job](https://jenkins.mc-stan.org/job/BuildDocs/)"}' "https://api.github.com/repos/stan-dev/docs/pulls"
                    """
                }
            }
        }
        stage('Clean checkout for cmdstan') {
            steps {
                deleteDir()
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/master']],
                          doGenerateSubmoduleConfigurations: false,
                          extensions: [],
                          submoduleCfg: [],
                          userRemoteConfigs: [[url: "https://github.com/stan-dev/cmdstan.git", credentialsId: 'a630aebc-6861-4e69-b497-fd7f496ec46b']]]
                )
            }
        }
        stage("Build cmdstan manual") {
            steps{
                sh "make manual"
                archiveArtifacts 'doc/*.pdf'
            }
        } 
    }
}

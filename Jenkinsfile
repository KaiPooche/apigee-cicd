#!groovy

//@Library('slackNotifications-shared-library@master') _

pipeline {
    agent any
    tools {
        maven 'maven'
        jdk 'JDK'
        nodejs 'NODEJS'
    }

    environment {
        //getting the current stable/deployed revision...this is used in undeloy.sh in case of failure...
        stable_revision = sh(script: 'curl -H "Authorization: Bearer ya29.a0AfB_byAA_8mhOHEV50gA77bDH2sdjvdsOrOYFTsMyOnSG3XML92_ZVAQgcyN7gwQfuX8c9-gfjcTKDFBBK9AjzksXy7F4CGBImpNRwJL5zQ5-IT3nzpxegMvVOv-o_-Yx1DnAh_UirFYfbUwwTHgth8nYziHlZstsKmwG6gxb2I_MMUlKj8-LqZzjCNXIgQcO4g295AhyeOw-ebcFuOCYRde-DaN1Mz4MwbWUWUtPrZSgOlPoPpBg6UWACe6HSpgAyNsn4enD1OeYqni0NZmmDR9Wp3aoKp-6NtjluD9JQnMpC05FOy2rJ94EYzW8cg0YZFC11n51-PuNMoBtWCgfIExyr6-sVlWtP0vu4FSZBadezUK78i1MxJ3V9ufJ4qBFo2Uau-PLLuHh7FQ4aEqPS4aCgYKAckSARMSFQHsvYlsPbIRCfehNn18fayovd4SCQ0414" "https://apigee.googleapis.com/v1/organizations/inductive-cocoa-390420/apis/HR-API/deployments" | jq -r ".environment[0].revision[0].name"', returnStdout: true).trim()
    }

    stages {
        stage('Initial-Checks') {
            steps {
                //sendNotifications 'STARTED'
                sh "npm -v"
                sh "mvn -v"
                echo "$apigeeUsername"
                echo "Stable Revision: ${env.stable_revision}"
        }}  
        stage('Policy-Code Analysis') {
            steps {
                sh "npm install -g apigeelint"
                sh "apigeelint -s HR-API/apiproxy/ -f codeframe.js"
            }
        }
        stage('Unit-Test-With-Coverage') {
            steps {
                script {
                    try {
                        sh "npm install"
                        sh "npm test test/unit/*.js"
                        sh "npm run coverage test/unit/*.js"
                    } catch (e) {
                        throw e
                    } finally {
                        sh "cd coverage && cp cobertura-coverage.xml $WORKSPACE"
                        step([$class: 'CoberturaPublisher', coberturaReportFile: 'cobertura-coverage.xml'])
                    }
                }
            }
        }
        /*stage('Promotion') {
            steps {
                timeout(time: 2, unit: 'DAYS') {
                    input 'Do you want to Approve?'
                }
            }
        }*/
        stage('Deploy to Production') {
            steps {
                 //deploy using maven plugin
                 
                 // deploy only proxy and deploy both proxy and config based on edge.js update
                //bat "sh && sh deploy.sh"
                sh "mvn -f HR-API/pom.xml install -Pprod -Dusername=${apigeeUsername} -Dpassword=${apigeePassword} -Dapigee.config.options=update"
            }
        }
        stage('Integration Tests') {
            steps {
                script {
                    try {
                        // using credentials.sh to get the client_id and secret of the app..
                        // thought of using them in cucumber oauth feature
                        // bat "sh && sh credentials.sh"
                        sh "cd $WORKSPACE/test/integration && npm install"
                        sh "cd $WORKSPACE/test/integration && npm test"
                    } catch (e) {
                        //if tests fail, I have used an shell script which has 3 APIs to undeploy, delete current revision & deploy previous stable revision
                        sh "sh && sh undeploy.sh"
                        throw e
                    } finally {
                        // generate cucumber reports in both Test Pass/Fail scenario
                        sh "cd $WORKSPACE/test/integration && cp reports.json $WORKSPACE"
                        cucumber fileIncludePattern: 'reports.json'
                        build job: 'cucumber-report'
                    }
                }
            }
        }
    }

    post {
        always {
            // cucumberSlackSend channel: 'apigee-cicd', json: '$WORKSPACE/reports.json'
             echo 'Hello world!'
           // sendNotifications currentBuild.result
        }
    }
}

/*

using shared library for slack reporting
    the lib groovy script must be placed in a vars folder in SCM
using build job to call a Freestyle project which sends the Cucumber reports to slack
    currently the cucumberSlackSend channel: 'apigee-cicd', json: '$WORKSPACE/reports.json' 
        option doesnt send the reports to Slack
*/

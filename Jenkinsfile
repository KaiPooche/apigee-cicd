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
        stable_revision = sh(script: 'curl -H "Authorization: Bearer ya29.a0AfB_byCP7orPXT6BKg5eVY73b3kQ1eH5AbOlJdufeUwQO2fptPBXRW33QH21cZZ6nqqRiqbLjXiWw8gywVe8tJapkc29tLULte7qyFkF5kQvpbhqsFQfOaGDYkMZ79RmXYYTv47EOSsq_V9G3omyI09CsQx6GNcDKcjZOgyny-MUeOwoFYMMb5ekvV4ef7d12Yasyl6qpmg7em4EFNFx9Lle6nffHpLu_ARxE3M_-cYfcSkHrBvgvARY495jrJcWWudp644YNavv4LiuQr4vIU-jRS6mId7NnIw4KXbqBKlPe4SV9Oh2oHNzDncaKvVADiUx-1fNuvp56QPv5ED9AtHaQ2lHbvB6IGIqBU9feHl2SkuJjauXccP_nJb-3VE5OLxJTorGMNfAu_Os0b7Bo6EaCgYKAegSARMSFQHsvYls7MFr_3e02_9hFdI1igw-3A0414" "https://apigee.googleapis.com/v1/organizations/inductive-cocoa-390420/apis/common-template/deployments" | jq -r ".environment[0].revision[0].name"', returnStdout: true).trim()
    }

    stages {
        stage('Initial-Checks') {
            steps {
                //sendNotifications 'STARTED'

                echo "using new config"
                 echo "$JAVA_HOME"
                sh "npm -v"
                sh "mvn -v"
               //echo "$apigeeUsername"
                //echo "Stable Revision: ${env.stable_revision}"
        }}  
        stage('Policy-Code Analysis') {
            steps {
                sh "npm install -g apigeelint"
                sh "apigeelint -s hr-api/apiproxy/ -f codeframe.js"
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

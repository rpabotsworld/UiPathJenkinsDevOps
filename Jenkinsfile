/*
    This is an example pipeline that implement full CI/CD for a UiPath Based Project.
    The pipeline is made up of 6 main steps
    1. Git clone and setup
    2. Build and local tests
    3. Publish Docker and Helm
    4. Deploy to dev and test
    5. Deploy to staging and test
    6. Optionally deploy to production and test
*/


pipeline {
        agent any

            // Environment Variables
            environment {
            MAJOR = '1'
            MINOR = '1'
            //Orchestrator Services
            UIPATH_ORCH_URL = "https://cloud.uipath.com/"
            UIPATH_ORCH_LOGICAL_NAME = "rpabotsworld"
            UIPATH_ORCH_TENANT_NAME = "DEV"
            UIPATH_ORCH_FOLDER_NAME = "Shared"
            EMAIL_RECIPIENTS = "prasadsatish@outlook.com"
        }


        // Pipeline stages
        stages {
            // Printing Basic Information
            ////////// Step 1 //////////
            stage('Preparing'){
                steps {
                    echo "Jenkins Home ${env.JENKINS_HOME}"
                    echo "Jenkins URL ${env.JENKINS_URL}"
                    echo "Jenkins JOB Number ${env.BUILD_NUMBER}"
                    echo "Jenkins JOB Name ${env.JOB_NAME}"
                    echo "GitHub BranchName ${env.BRANCH_NAME}"
                    echo "Jenkins JOB URL ${env.JOB_URL}" 
                    echo "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} by ${env.BUILD_USER}\n More info at: ${env.BUILD_URL}"
                    checkout scm
                }
            }

            // Building Tests
            ////////// Step 2 //////////
            stage('Build Tests') {
                steps {
                    echo "Building package with ${WORKSPACE}"
                    UiPathPack (
                          outputPath: "Output\\Tests\${env.BUILD_NUMBER}",
                          outputType: 'Tests',
                          projectJsonPath: "project.json",
                          version: [$class: 'ManualVersionEntry', version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"],
                          useOrchestrator: false,
                          traceLevel: 'Verbose'
                        )
                }
            }

            // Create Assets 
            ////////// Step 3 //////////
            stage ('Create Assests') {
                steps {
                    echo "Creating Assests with ${WORKSPACE}"
                    UiPathAssets (
                        assetsAction: DeployAssets(), 
                        filePath: "${WORKSPACE}\\Data\\Assests.csv", 
                        orchestratorAddress: "${UIPATH_ORCH_URL}",
                        orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                        folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                        //credentials: [$class: 'UserPassAuthenticationEntry', credentialsId: 'APIUserKey']
                        credentials: Token(accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'APIUserKey'),
                        traceLevel: 'Verbose'
                    )
       
                }
            }

            // Deploy Stages
            ////////// Step 4 //////////
            stage('Deploy Tests') {
                steps {
                    echo "Deploying ${BRANCH_NAME} to orchestrator"
                    UiPathDeploy (
                    packagePath: "Output\\Tests\${env.BUILD_NUMBER}",
                    orchestratorAddress: "${UIPATH_ORCH_URL}",
                    orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                    folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                    environments: 'INT',
                    //credentials: [$class: 'UserPassAuthenticationEntry', credentialsId: 'APIUserKey']
                    credentials: Token(accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'APIUserKey'),
                    traceLevel: 'Verbose',
                    entryPointPaths: 'Main.xaml'
                    )
                }
            
            }
            
            
            //  // Test Stages
            ////////// Step 5 //////////
            // stage('Perform Tests') {
            //     steps {
            //        echo 'Testing the workflow...'
            //         UiPathTest (
            //           testTarget: [$class: 'TestSetEntry', testSet: "UiPathJenkinsDevOps_Tests"],
            //           orchestratorAddress: "${UIPATH_ORCH_URL}",
            //           orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
            //           folderName: "${UIPATH_ORCH_FOLDER_NAME}",
            //           timeout: 10000,
            //           traceLevel: 'Information',
            //           testResultsOutputPath:"Output\\Tests\\${env.BUILD_NUMBER}\\result.xml",
            //           //credentials: [$class: 'UserPassAuthenticationEntry', credentialsId: "credentialsId"]
            //           credentials: Token(accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'APIUserKey'),
            //           parametersFilePath: ''
            //         )
            //     }
            // }

            //////////// Step 6 //////////
            stage('Development deploy approval and deployment') {
                steps {
                    script {
                        if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
                            timeout(time: 3, unit: 'MINUTES') {
                                // you can use the commented line if u have specific user group who CAN ONLY approve
                                //input message:'Approve deployment?', submitter: 'it-ops'
                                input message: 'Approve deployment?'
                            }
                        }
                    }
                }
            }

            //////////// Step 7 //////////
            stage("Approval") {
                steps{
                 script {
                    echo "====Waiting for Approval===="
                    emailext mimeType: 'text/html',
                             subject: "[Jenkins-Deploy-Approval]${currentBuild.fullDisplayName}",
                             to: "univmentor@gmail.com",
                             attachLog: true,
                             body: """<!DOCTYPE html>
                                   <html>
                                   <head> 
                                      <style>
                                         #customers td, #customers th {
                                             border: 1px solid black;
                                             padding: 6px;
                                             border-collapse: collapse;
                                             }
                                          #tableheader th {
                                              font-weight: bold;
                                              }
                                          .footer {
                                              position: fixed;
                                              left: 30;
                                              right: 60;
                                              bottom: 20;
                                              width: 20%;
                                              color: black;
                                              text-align: left;
                                              }
                                        body{        
                                            padding-bottom: 400px;
                                            }
                                        </style>
                                        </head>
                                        <body>
                                        <table>
                                          <tr style="background-color:white;color:black;">
                                             <th width="10"><img src="http://i.imgur.com/uXlqCxW.gif" alt="Smiley face" height="30" width="30"></th>
                                             <th align="left"><strong>BUILD APPROVAL</strong></th>
                                          </tr>
                                        </table>
                                        <p><strong>Build URL:</strong><a href="${env.BUILD_URL}input">click to approve</a></p>
                                        <p><strong>Project:</strong> ${currentBuild.fullDisplayName}</p>
                                        <p><strong>Project:</strong> ${env.JOB_NAME}</p>
                                    </body>
                                    <div class="footer">
                                    <footer>
                                      <p><strong>thanks</strong></p>
                                      <p>DevOps Team</p>
                                    </footer>
                                    </div>
                                    </html>"""
                    def userInput = input id: 'userInput',
                              message: 'Let\'s promote?', 
                              submitterParameter: 'submitter',
                              submitter: 'admin',
                              parameters: [
                                  [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Environment', name: 'DEPLOY_TO_PROD'],
                                  [$class: 'TextParameterDefinition', defaultValue: 'k8s', description: 'Target', name: 'target']]
                    
                    echo ("DEPLOY_TO_PROD: "+userInput['DEPLOY_TO_PROD'])
                    echo ("Target: "+userInput['target'])
                    echo ("submitted by: "+userInput['submitter'])

                    env.DEPLOY_TO_PROD = userInput.DEPLOY_TO_PROD
                    env.BUILD_APPROVED = userInput.submitter
                    echo "Selected Environment: ${DEPLOY_TO_PROD}"
                    }
                }
            }


             // Building Package
            stage('Build Process') {
                when {
                    expression {
                        currentBuild.result == null || currentBuild.result == 'SUCCESS'
                        }
                }
                steps {
                    echo "Building package with ${WORKSPACE}"
                    UiPathPack (
                          outputPath: "Output\\${env.BUILD_NUMBER}",
                          projectJsonPath: "project.json",
                          version: [$class: 'ManualVersionEntry', version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"],
                          useOrchestrator: false,
                          traceLevel: 'Verbose'
                        )
                    }
            }           
            
             // Deploy to UAT Step
            stage('Deploy Process') {
                when {
                    expression {
                        currentBuild.result == null || currentBuild.result == 'SUCCESS' 
                        }
                }
                steps {
                    echo 'Deploying process to orchestrator...'
                    UiPathDeploy (
                    packagePath: "Output\\${env.BUILD_NUMBER}",
                    orchestratorAddress: "${UIPATH_ORCH_URL}",
                    orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                    folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                    environments: 'INT',
                    //credentials: [$class: 'UserPassAuthenticationEntry', credentialsId: 'APIUserKey']
                    credentials: Token(accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'APIUserKey'),
                    traceLevel: 'Verbose',
                    entryPointPaths: 'Main.xaml'
                    )
                }   
            }   
        
        }

        // 
        post {
            success {
                echo 'Deployment has been completed!'
                sendEmail("Successful");
            }
            failure {
              echo "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.JOB_DISPLAY_URL})"
              sendEmail("Failed");
            }
            always {
                /* Clean workspace if success */
                cleanWs()
            }
        }
    
    }

def sendEmail(status) {
    mail(
            to: "$EMAIL_RECIPIENTS",
            subject: "Build $BUILD_NUMBER - " + status + " (${currentBuild.fullDisplayName})",
            body: "Changes:\n " + " " + "\n\n Check console output at: $BUILD_URL/console" + "\n")
}

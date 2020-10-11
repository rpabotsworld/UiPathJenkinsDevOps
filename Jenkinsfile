pipeline {
    agent any

        // Environment Variables
        environment {
        ENV = "DEV"
        MAJOR = '1'
        MINOR = '0'
        //Orchestrator Services
        UIPATH_ORCH_URL = "https://cloud.uipath.com/AIFabricDemo/"
        UIPATH_ORCH_LOGICAL_NAME = "AIFabricDemo"
        UIPATH_ORCH_TENANT_NAME ="DEV45qc611008"

  }

    stages {

         // Build Stages
        stage('Build') {
            steps {
                echo "Building..with ${WORKSPACE}"
                UiPathPack (
                      outputPath: "Output\\${env.BUILD_NUMBER}",
                      projectJsonPath: "project.json",
                      version: [$class: 'ManualVersionEntry', version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"],
                      useOrchestrator: false
        )
            }
        }
         // Test Stages
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }

         // Deploy Stages
        stage('Deploy to UAT') {
            steps {
                echo "Deploying ${BRANCH_NAME}"
            }
        }


         // Deploy to Production Step
        stage('Deploy to Production') {
            when {
                allOf {
                    branch 'master'
                    expression { return DPROD ==~ /(?i)(Y|YES|T|TRUE|ON|RUN)/ }
                }
            }
            steps {
                echo 'Deploy to Production'
                
            }
        }
    }

    // Options
    options {
        // Timeout for pipeline
        timeout(time:80, unit:'MINUTES')
        skipDefaultCheckout()
    }


    // 
    post {
        success {
            echo 'Deployment has been completed!'
        }
        failure {
          echo "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.JOB_DISPLAY_URL})"
        }
        always {
            /* Clean workspace if success */
            cleanWs()
        }
    }

}

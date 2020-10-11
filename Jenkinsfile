pipeline {
    agent any

        environment {
        ENV = "DEV"
        ORCH_URL = "https://deployment.example.com/"
        MAJOR = '1'
        MINOR = '0'
  }
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
                UiPathPack (
                      outputPath: "Output\\${env.BUILD_NUMBER}",
                      projectJsonPath: "UiPathJenkinsDevOps\\project.json",
                      version: [$class: 'ManualVersionEntry', version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"],
                      useOrchestrator: false
        )
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }

}

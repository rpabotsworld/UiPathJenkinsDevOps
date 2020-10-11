pipeline {
    agent any

        environment {
        ENV = "DEV"
        ORCH_URL = "https://deployment.example.com/"
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
                UiPathPack outputPath: '${WORKSPACE}\\Output', outputType: '', projectJsonPath: '${WORKSPACE}'
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

pipeline {
    agent any
    stages {
        stage('Check Node.js Installation') {
            steps {
                script {
                    def nodeInstalled = sh(
                        script: 'node --version || echo "not installed"',
                        returnStdout: true
                    ).trim()

                    if (nodeInstalled == "not installed") {
                        error("Node.js is not installed. Please install Node.js manually before running the pipeline.")
                    } else {
                        echo "Node.js is installed: ${nodeInstalled}"
                    }
                }
            }
        }
        stage('Install Dependencies') { 
            steps {
                sh 'npm install' 
            }
        }
        stage('Run Application') {
            steps {
                sh 'npm start' // Adjust the start script according to your application setup
            }
        }
    }
}

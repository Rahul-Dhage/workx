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
                        echo "Node.js is not installed. Installing Node.js..."
                        sh '''
                            # Install Node.js (example for Ubuntu/Debian)
                            curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
                            sudo apt-get install -y nodejs
                        '''
                    } else {
                        echo "Node.js is already installed: ${nodeInstalled}"
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

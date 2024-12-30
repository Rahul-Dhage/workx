pipeline {
    agent any
    
    environment {
        PATH = "/usr/local/bin:${env.PATH}"
        NVM_DIR = "/var/lib/jenkins/.nvm"
        PORT = "6969"
    }

    stages {
        stage('Setup Node.js') {
            steps {
                sh '''
                    if [ ! -d "$NVM_DIR" ]; then
                        curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
                        . "$NVM_DIR/nvm.sh"
                        nvm install 18
                        nvm use 18
                    fi
                    
                    # Source NVM
                    . "$NVM_DIR/nvm.sh"
                    
                    # Verify installations
                    node --version
                    npm --version
                '''
            }
        }
        
        stage('Check & Install Dependencies') {
            steps {
                script {
                    def nodeModulesExists = fileExists 'node_modules'
                    if (!nodeModulesExists) {
                        echo 'node_modules not found. Installing dependencies...'
                        sh '. "$NVM_DIR/nvm.sh" && npm install'
                    } else {
                        echo 'node_modules found, skipping npm install'
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                sh '. "$NVM_DIR/nvm.sh" && npm run build'
            }
        }

        stage('Deploy & Run') {
            steps {
                script {
                    // Kill any existing process on PORT 6969 if it exists
                    sh 'npx kill-port 6969 || true'
                    
                    // Start the application
                    sh '''
                        . "$NVM_DIR/nvm.sh"
                        nohup npm run preview > app.log 2>&1 &
                        echo "Application started on port ${PORT}"
                        sleep 10
                        curl http://localhost:${PORT} || echo "Application may need more time to start"
                    '''
                }
            }
        }
    }
}
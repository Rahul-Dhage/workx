pipeline {
    agent any
    
    environment {
        PATH = "/usr/local/bin:${env.PATH}"
        NVM_DIR = "/var/lib/jenkins/.nvm"
        PORT = "6000"
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
                    
                    # Install PM2 globally if not installed
                    npm install pm2 -g || true
                    
                    # Verify installations
                    node --version
                    npm --version
                    pm2 --version
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
                    sh '''
                        # Source NVM
                        . "$NVM_DIR/nvm.sh"
                        
                        # Stop any existing PM2 process
                        pm2 delete vite-app || true
                        
                        # Start the application with PM2
                        pm2 start npm --name "vite-app" -- run preview
                        
                        # Save PM2 process list
                        pm2 save
                        
                        # Display process status
                        pm2 list
                        
                        echo "Waiting for application to start..."
                        sleep 10
                        
                        # Check if the application is running
                        if curl -s http://localhost:6000 > /dev/null; then
                            echo "Application is running successfully"
                            pm2 logs vite-app --lines 10
                        else
                            echo "Application failed to start"
                            pm2 logs vite-app --lines 50
                            exit 1
                        fi
                    '''
                }
            }
        }
    }

    post {
        always {
            script {
                sh '''
                    echo "Process Status:"
                    ps aux | grep node
                    echo "Port Status:"
                    netstat -tulpn | grep 6000 || true
                    echo "PM2 Status:"
                    pm2 list || true
                '''
            }
        }
    }
}
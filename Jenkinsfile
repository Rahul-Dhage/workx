pipeline {
    agent any
    
    environment {
        PATH = "/usr/local/bin:${env.PATH}"
        NVM_DIR = "/var/lib/jenkins/.nvm"
    }

    stages {
        stage('Setup Node.js') {
            steps {
                sh '''
                    # Install NVM if not present
                    if [ ! -d "$NVM_DIR" ]; then
                        curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
                        . "$NVM_DIR/nvm.sh"
                        nvm install 18
                        nvm use 18
                    fi
                    
                    # Add NVM to PATH
                    export NVM_DIR="$HOME/.nvm"
                    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
                    
                    # Verify installations
                    node --version
                    npm --version
                '''
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh '''
                    . "$NVM_DIR/nvm.sh"
                    npm install
                '''
            }
        }
        
        stage('Build') {
            steps {
                sh '''
                    . "$NVM_DIR/nvm.sh"
                    npm run build
                '''
            }
        }
    }
}
pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'react-app'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Create Configuration Files') {
            steps {
                script {
                    // Create nginx.conf
                    writeFile file: 'nginx.conf', text: '''
server {
    listen 5173;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
}
'''
                    // Create Dockerfile
                    writeFile file: 'Dockerfile', text: '''
# Build stage
FROM node:18-alpine as build

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine

# Copy built assets
COPY --from=build /app/dist /usr/share/nginx/html

# Copy nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 5173

CMD ["nginx", "-g", "daemon off;"]
'''
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                        # Build Docker image
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        
                        # Stop existing container if running
                        docker stop ${DOCKER_IMAGE} || true
                        docker rm ${DOCKER_IMAGE} || true
                        
                        # Run new container
                        docker run -d \
                            --name ${DOCKER_IMAGE} \
                            -p 5173:5173 \
                            --restart unless-stopped \
                            ${DOCKER_IMAGE}:${DOCKER_TAG}
                            
                        # Check container status
                        docker ps | grep ${DOCKER_IMAGE}
                    '''
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                script {
                    sh '''
                        # Wait for container to start
                        sleep 15
                        
                        # Check if service is responding
                        curl -s http://localhost:5173 || echo "Service not responding"
                        
                        # Show container logs
                        docker logs ${DOCKER_IMAGE}
                        
                        # Get container IP
                        echo "Container IP and Port Information:"
                        docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ${DOCKER_IMAGE}
                        docker port ${DOCKER_IMAGE}
                        
                        # Print access instructions
                        echo "Application should be accessible at:"
                        echo "Local: http://localhost:5173"
                        echo "Remote: http://$(hostname -I | awk '{print $1}'):5173"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful! Application is running in Docker container.'
        }
        failure {
            script {
                sh '''
                    echo "Deployment failed. Collecting diagnostics..."
                    docker logs ${DOCKER_IMAGE} || true
                    docker ps -a
                    netstat -tulpn | grep 5173 || true
                '''
            }
        }
        always {
            script {
                sh '''
                    echo "=== Deployment Status ==="
                    echo "Docker Container Status:"
                    docker ps -a | grep ${DOCKER_IMAGE} || true
                    echo "Port Status:"
                    netstat -tulpn | grep 5173 || true
                '''
            }
        }
    }
}
pipeline {
    agent any
    environment {
        VENV_DIR = 'venv'
        APP_HOST = '0.0.0.0'
        APP_PORT = '8000'
        SERVICE_NAME = 'fastapi-app'  // Change to your actual service name
        APP_DIR = '/path/to/your/app'  // Change to your actual app directory
    }
    stages {
        stage('Checkout') {
            steps {
                echo '📦 Checking out code from GitHub...'
                git branch: 'main', url: 'https://github.com/mohammad-junaid79/jenkins_CI-CD.git'
            }
        }
        stage('Setup Python Environment') {
            steps {
                echo '🐍 Setting up Python virtual environment...'
                sh '''
                    if [ ! -d "$VENV_DIR" ]; then
                        python3 -m venv $VENV_DIR
                    fi
                    . $VENV_DIR/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }
        stage('Stop Service') {
            steps {
                echo '🛑 Stopping FastAPI service...'
                sh '''
                    # Stop the systemd service
                    sudo systemctl stop $fastapi || true
                    
                    # Kill any remaining processes
                    pkill -f "uvicorn main:app" || true
                    
                    # Wait for port to be free
                    sleep 3
                '''
            }
        }
        stage('Deploy Code') {
            steps {
                echo '📂 Deploying updated code...'
                sh '''
                    # Optional: backup old code
                    # cp -r $APP_DIR ${APP_DIR}_backup_$(date +%Y%m%d_%H%M%S)
                    
                    echo "✅ Code updated in workspace"
                '''
            }
        }
        stage('Start Service') {
            steps {
                echo '🚀 Starting FastAPI service...'
                sh '''
                    # Start the systemd service
                    sudo systemctl start $fastapi
                    
                    # Wait for service to start
                    sleep 5
                    
                    # Check service status
                    sudo systemctl status $fastapi --no-pager
                '''
            }
        }
        stage('Verify Deployment') {
            steps {
                echo '🔍 Verifying FastAPI is running...'
                sh '''
                    # Wait a bit more for the service to be fully ready
                    sleep 3
                    
                    # Check if service is active
                    if ! sudo systemctl is-active --quiet $fastapi; then
                        echo "❌ Service is not active!"
                        exit 1
                    fi
                    
                    # Test the endpoint
                    curl -f http://$APP_HOST:$APP_PORT || (echo "❌ FastAPI is not responding!" && exit 1)
                    echo "✅ FastAPI is running successfully!"
                '''
            }
        }
    }
    post {
        success {
            echo '✅ Pipeline completed successfully!'
            echo '🚀 FastAPI running: http://192.168.35.35:8000'
            echo '📖 API docs: http://192.168.35.35:8000/docs'
        }
        failure {
            echo '❌ Pipeline failed! Check the logs for details.'
            sh '''
                echo "📋 Service logs:"
                sudo journalctl -u $fastapi -n 50 --no-pager
            '''
        }
    }
}

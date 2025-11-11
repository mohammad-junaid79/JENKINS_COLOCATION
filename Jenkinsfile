pipeline {
    agent {
        label 'deployment'  // Uses your deployment server node
    }
    
    environment {
        VENV_DIR = '/data/10nov2025/app/venv'
        APP_DIR = '/data/10nov2025/app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '📦 Checking out code from GitHub...'
                git branch: 'main', url: 'https://github.com/mohammad-junaid79/JENKINS_COLOCATION.git'
            }
        }
        
        stage('Setup Python Environment') {
            steps {
                echo '🐍 Setting up Python virtual environment...'
                sh '''
                    # Create app directory if not exists
                    mkdir -p $APP_DIR
                    
                    # Create virtual environment using existing Python 3.10.12
                    if [ ! -d "$VENV_DIR" ]; then
                        python3 -m venv $VENV_DIR
                    fi
                    
                    # Activate and install dependencies
                    . $VENV_DIR/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }
        
        stage('Deploy Code') {
            steps {
                echo '📂 Deploying code to application directory...'
                sh '''
                    # Copy application files to app directory
                    cp -r main.py requirements.txt $APP_DIR/
                    
                    echo "✅ Code deployed to: $APP_DIR"
                    ls -lah $APP_DIR
                '''
            }
        }
        
        stage('Restart FastAPI Service') {
            steps {
                echo '🔄 Restarting FastAPI service...'
                sh '''
                    sudo systemctl restart fast.service
                    sleep 3
                '''
            }
        }
        
        stage('Verify Deployment') {
            steps {
                echo '🔍 Verifying FastAPI is running...'
                sh '''
                    # Check if service is active
                    sudo systemctl is-active --quiet fast.service || (echo "❌ FastAPI service is not running!" && exit 1)
                    
                    # Test the endpoint
                    curl -f http://localhost:8000 || (echo "❌ FastAPI is not responding!" && exit 1)
                    
                    echo "✅ FastAPI is running successfully!"
                '''
            }
        }
    }
    
    post {
        success {
            echo '========================================='
            echo '✅ Pipeline completed successfully!'
            echo '========================================='
            echo '🚀 FastAPI Application Status:'
            echo '   - Service: Running'
            echo '   - URL: http://192.168.35.25:8000'
            echo '   - API Docs: http://192.168.35.25:8000/docs'
            echo '========================================='
        }
        failure {
            echo '❌ Pipeline failed! Check the logs for details.'
            echo 'Common issues:'
            echo '  - Check if GitHub repo is accessible'
            echo '  - Verify requirements.txt exists'
            echo '  - Check FastAPI service logs: sudo journalctl -u fast.service -n 50'
        }
    }
}

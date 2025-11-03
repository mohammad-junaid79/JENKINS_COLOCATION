pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from GitHub...'
                git branch: 'main', url: 'https://github.com/mohammad-junaid79/jenkins_CI-CD.git'
            }
        }
        
        stage('Setup Python Environment') {
            steps {
                echo 'Setting up Python virtual environment...'
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }
        
        stage('Stop Old Instance') {
            steps {
                echo 'Stopping old FastAPI instance...'
                sh '''
                    if [ -f fastapi.pid ]; then
                        kill $(cat fastapi.pid) || true
                        rm fastapi.pid
                    fi
                    pkill -f "uvicorn main:app" || true
                '''
            }
        }
        
        stage('Deploy FastAPI') {
            steps {
                echo 'Starting FastAPI application...'
                sh '''
                    . venv/bin/activate
                    nohup uvicorn main:app --host 0.0.0.0 --port 8000 > fastapi.log 2>&1 &
                    echo $! > fastapi.pid
                    sleep 5
                '''
            }
        }
        
        stage('Verify Deployment') {
            steps {
                echo 'Verifying FastAPI is running...'
                sh '''
                    curl -f http://localhost:8000 || exit 1
                    echo "FastAPI is running successfully!"
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
        echo '❌ Pipeline failed! Check logs for details.'
    }
}

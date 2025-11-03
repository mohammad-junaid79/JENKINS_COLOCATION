pipeline {
    agent any

    environment {
        VENV_DIR = 'venv'
        APP_HOST = '0.0.0.0'
        APP_PORT = '8000'
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

        stage('Stop Old Instance') {
            steps {
                echo '🛑 Stopping old FastAPI instance if running...'
                sh '''
                    if [ -f fastapi.pid ]; then
                        kill $(cat fastapi.pid) || true
                        rm -f fastapi.pid
                    fi

                    pkill -f "uvicorn main:app" || true
                '''
            }
        }

        stage('Deploy FastAPI') {
            steps {
                echo '🚀 Starting FastAPI application...'
                sh '''
                    . $VENV_DIR/bin/activate
                    nohup uvicorn main:app --host $APP_HOST --port $APP_PORT > fastapi.log 2>&1 &
                    echo $! > fastapi.pid
                    sleep 5
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo '🔍 Verifying FastAPI is running...'
                sh '''
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
        }
    }
}

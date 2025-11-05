pipeline {
    agent any
    environment {
        VENV_DIR = 'venv'
        APP_HOST = '0.0.0.0'
        APP_PORT = '8000'
        APP_DIR = '/var/lib/jenkins/workspace/fastapi_colocation'
        PID_FILE = '/var/lib/jenkins/workspace/fastapi_colocation/app.pid'
        LOG_FILE = '/var/lib/jenkins/workspace/fastapi_colocation/app.log'
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
        
        stage('Stop Old Process') {
            steps {
                echo '🛑 Stopping old FastAPI process...'
                sh '''
                    # Stop by PID file if it exists
                    if [ -f "$PID_FILE" ]; then
                        OLD_PID=$(cat $PID_FILE)
                        if ps -p $OLD_PID > /dev/null 2>&1; then
                            echo "Killing process $OLD_PID"
                            kill $OLD_PID || true
                            sleep 2
                            # Force kill if still running
                            if ps -p $OLD_PID > /dev/null 2>&1; then
                                kill -9 $OLD_PID || true
                            fi
                        fi
                        rm -f $PID_FILE
                    fi
                    
                    # Kill any uvicorn processes on port 8000
                    pkill -f "uvicorn main:app" || true
                    
                    # Wait for port to be free
                    sleep 3
                    
                    # Verify port is free
                    if lsof -Pi :8000 -sTCP:LISTEN -t >/dev/null 2>&1; then
                        echo "⚠️ Port 8000 still in use, force killing..."
                        lsof -ti:8000 | xargs kill -9 || true
                        sleep 2
                    fi
                    
                    echo "✅ Old process stopped, port 8000 is free"
                '''
            }
        }
        
        stage('Deploy Code') {
            steps {
                echo '📂 Deploying updated code...'
                sh '''
                    echo "✅ Code updated in workspace: $APP_DIR"
                    ls -lah $APP_DIR
                '''
            }
        }
        
        stage('Start Service with nohup') {
            steps {
                echo '🚀 Starting FastAPI with nohup...'
                sh '''
                    cd $APP_DIR
                    
                    # Activate virtual environment
                    . $VENV_DIR/bin/activate
                    
                    # CRITICAL: Use BUILD_ID=dontKillMe to prevent Jenkins from killing the process
                    BUILD_ID=dontKillMe nohup uvicorn main:app \
                        --host $APP_HOST \
                        --port $APP_PORT \
                        > $LOG_FILE 2>&1 &
                    
                    # Save PID to file
                    echo $! > $PID_FILE
                    
                    echo "✅ Started uvicorn with PID: $(cat $PID_FILE)"
                    
                    # Wait for startup
                    sleep 5
                '''
            }
        }
        
        stage('Verify Deployment') {
            steps {
                echo '🔍 Verifying FastAPI is running...'
                sh '''
                    # Check if PID file exists
                    if [ ! -f "$PID_FILE" ]; then
                        echo "❌ PID file not found!"
                        exit 1
                    fi
                    
                    # Check if process is running
                    PID=$(cat $PID_FILE)
                    if ! ps -p $PID > /dev/null 2>&1; then
                        echo "❌ Process $PID is not running!"
                        echo "📋 Last 20 lines of log:"
                        tail -20 $LOG_FILE
                        exit 1
                    fi
                    
                    echo "✅ Process $PID is running"
                    
                    # Wait a bit more for the service to be fully ready
                    sleep 3
                    
                    # Test the endpoint (retry up to 5 times)
                    for i in 1 2 3 4 5; do
                        if curl -f -s http://$APP_HOST:$APP_PORT > /dev/null; then
                            echo "✅ FastAPI is responding!"
                            curl http://$APP_HOST:$APP_PORT
                            exit 0
                        fi
                        echo "Attempt $i failed, retrying..."
                        sleep 2
                    done
                    
                    echo "❌ FastAPI is not responding after 5 attempts!"
                    echo "📋 Last 30 lines of log:"
                    tail -30 $LOG_FILE
                    exit 1
                '''
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline completed successfully!'
            echo '🚀 FastAPI running: http://192.168.35.35:8000'
            echo '📖 API docs: http://192.168.35.35:8000/docs'
            sh '''
                echo "📊 Process info:"
                ps aux | grep "[u]vicorn main:app"
                echo ""
                echo "📍 PID: $(cat $PID_FILE 2>/dev/null || echo 'N/A')"
                echo "📝 Logs: $LOG_FILE"
            '''
        }
        failure {
            echo '❌ Pipeline failed! Check the logs for details.'
            sh '''
                echo "📋 Application logs (last 50 lines):"
                if [ -f "$LOG_FILE" ]; then
                    tail -50 $LOG_FILE
                else
                    echo "Log file not found"
                fi
                
                echo ""
                echo "🔍 Checking for running uvicorn processes:"
                ps aux | grep uvicorn || echo "No uvicorn processes found"
                
                echo ""
                echo "🔍 Checking port 8000:"
                lsof -i :8000 || echo "Port 8000 not in use"
            '''
        }
        always {
            echo '📊 Pipeline execution completed'
        }
    }
}

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
}
```

---

### **Step 6: Save the Pipeline**

1. Scroll down to the bottom of the page
2. Click the blue **"Save"** button

---

### **Step 7: Build the Pipeline**

After saving, you'll be taken to the pipeline's main page.

1. On the left sidebar, click **"Build Now"**
2. You'll see a build appear in the **"Build History"** section (bottom left)
3. The build will have a number like **#1**

---

### **Step 8: Watch the Build Progress**

**Option A: Watch the stages (Recommended):**
- You'll see colored boxes representing each stage
- Blue = Running
- Green = Success
- Red = Failed

**Option B: View detailed logs:**
1. Click on the build number (e.g., **#1**)
2. Click **"Console Output"** on the left
3. Watch the live logs scroll

---

## **What You Should See:**

### **Successful Build:**
```
Stage View:
[✓] Checkout
[✓] Setup Python Environment
[✓] Stop Old Instance
[✓] Deploy FastAPI
[✓] Verify Deployment

✅ Pipeline completed successfully!
🚀 FastAPI running: http://192.168.35.35:8000

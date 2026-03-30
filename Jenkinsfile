pipeline {
    agent any

    environment {
        VENV_PATH = "${WORKSPACE}/venv"
        APP_PATH = "${WORKSPACE}/app.py"
        PYTHON = "python3"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Setup Environment') {
            steps {
                sh '''
                    set -e
                    set -x
                    # Remove old virtual environment
                    rm -rf $VENV_PATH
                    # Create new virtual environment
                    $PYTHON -m venv $VENV_PATH
                    # Activate venv (POSIX-compliant)
                    . $VENV_PATH/bin/activate
                    # Upgrade pip and install dependencies
                    pip install --upgrade pip
                    if [ -f requirements.txt ]; then
                        pip install -r requirements.txt
                    fi
                '''
            }
        }

        stage('Stop Old App') {
            steps {
                sh '''
                    # Optional: stop old Flask/Sockets app if running
                    pkill -f "python.*app.py" || true
                '''
            }
        }

        stage('Start App') {
            steps {
                sh '''
                    . $VENV_PATH/bin/activate
                    nohup $PYTHON $APP_PATH > app.log 2>&1 &
                    echo "Flask app started in background, logs: app.log"
                '''
            }
        }
    }

    post {
        failure {
            echo "❌ Deployment failed. Check logs for details."
        }
        success {
            echo "✅ Deployment succeeded."
        }
    }
}

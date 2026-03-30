pipeline {
    agent any

    environment {
        APP_DIR = "${WORKSPACE}"
        VENV_DIR = "${WORKSPACE}/venv"
        PORT = "5000"
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
                    # Remove old virtual environment if exists
                    rm -rf $VENV_DIR

                    # Create new virtual environment
                    python3 -m venv $VENV_DIR

                    # Activate virtual environment
                    . $VENV_DIR/bin/activate

                    # Upgrade pip
                    pip install --upgrade pip

                    # Install dependencies
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Stop Old App') {
            steps {
                sh '''
                    OLD_PID=$(lsof -t -i:$PORT || true)
                    if [ -n "$OLD_PID" ]; then
                        kill -9 $OLD_PID
                        echo "Stopped old app (PID: $OLD_PID)"
                    else
                        echo "No old app running on port $PORT"
                    fi
                '''
            }
        }

        stage('Start App') {
            steps {
                sh '''
                    cd $APP_DIR
                    . $VENV_DIR/bin/activate

                    # Start Flask app in background
                    nohup python app.py --host=0.0.0.0 --port=5000 > flask.log 2>&1 &
                    echo "Flask app started on port $PORT"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful"
        }
        failure {
            echo "❌ Deployment Failed"
        }
    }
}

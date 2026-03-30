pipeline {
    agent any

    environment {
        APP_DIR = "/var/lib/jenkins/workspace/Flask-chat-app"
        VENV_DIR = "${APP_DIR}/venv"
        FLASK_LOG = "${APP_DIR}/flask.log"
        PORT = "5000"
    }

    stages {
        stage('Checkout Code') {
            steps {
                sshagent(['git']) {
                    sh """
                    cd $APP_DIR
                    git fetch --all
                    git reset --hard origin/main
                    """
                }
            }
        }

        stage('Setup Environment') {
            steps {
                sh """
                cd $APP_DIR
                # Remove old virtual environment if exists
                rm -rf $VENV_DIR
                # Create a new virtual environment
                python3 -m venv $VENV_DIR
                # Activate venv
                . $VENV_DIR/bin/activate
                # Upgrade pip and install dependencies
                pip install --upgrade pip
                pip install -r requirements.txt
                """
            }
        }

        stage('Stop Old App') {
            steps {
                sh """
                # Kill any process using the port
                OLD_PID=\$(lsof -t -i:$PORT)
                if [ -n "\$OLD_PID" ]; then
                    echo "Stopping old Flask app (PID \$OLD_PID)"
                    kill -9 \$OLD_PID
                else
                    echo "No old Flask app running on port $PORT"
                fi
                """
            }
        }

        stage('Start App') {
            steps {
                sh """
                cd $APP_DIR
                . $VENV_DIR/bin/activate
                echo "Starting Flask app on port $PORT"
                nohup python app.py --host=0.0.0.0 --port=$PORT > $FLASK_LOG 2>&1 &
                """
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

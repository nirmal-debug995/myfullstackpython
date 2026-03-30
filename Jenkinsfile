pipeline {
    agent any

    environment {
        APP_DIR = "/var/lib/jenkins/workspace/Flask-chat-app"
        VENV_DIR = "${APP_DIR}/venv"
        PORT = "5000"   // Flask default port
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Use your SSH credential ID
                sshagent(['github-ssh-key']) {
                    sh '''
                        # Clone repo if not exists
                        if [ ! -d "$APP_DIR" ]; then
                            git clone git@github.com:nirmal-debug995/myfullstackpython.git $APP_DIR
                        fi
                        cd $APP_DIR
                        git fetch --all
                        git reset --hard origin/main
                    '''
                }
            }
        }

        stage('Setup Environment') {
            steps {
                sh '''
                    cd $APP_DIR

                    # Remove old venv if exists
                    rm -rf $VENV_DIR

                    # Create new virtual environment
                    python3 -m venv $VENV_DIR

                    # Activate venv and install dependencies
                    . $VENV_DIR/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Stop Old App') {
            steps {
                sh '''
                    # Kill any old Flask process running on $PORT
                    OLD_PID=$(lsof -t -i:$PORT)
                    if [ ! -z "$OLD_PID" ]; then
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
                    # Activate venv and start Flask app in background
                    . $VENV_DIR/bin/activate
                    nohup python app.py > flask.log 2>&1 &
                    echo "Flask app started on port $PORT"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful!"
        }
        failure {
            echo "❌ Deployment Failed"
        }
    }
}

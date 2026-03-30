pipeline {
    agent any

    environment {
        APP_PORT = "5000"
        APP_DIR = "/var/lib/jenkins/workspace/Flask-chat-app"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'git@github.com:nirmal-debug995/myfullstackpython.git',
                        credentialsId: 'github-ssh-key'
                    ]]
                ])
            }
        }

        stage('Setup Environment') {
            steps {
                sh """
                    cd ${APP_DIR}
                    rm -rf venv
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                """
            }
        }

        stage('Stop Old App') {
            steps {
                sh """
                    OLD_PID=\$(lsof -t -i:${APP_PORT} || true)
                    if [ ! -z "\$OLD_PID" ]; then
                        kill -9 \$OLD_PID
                        echo "Stopped old app with PID \$OLD_PID"
                    else
                        echo "No old app running on port ${APP_PORT}"
                    fi
                """
            }
        }

        stage('Start App') {
            steps {
                sh """
                    cd ${APP_DIR}
                    . venv/bin/activate
                    nohup python app.py --host=0.0.0.0 --port=${APP_PORT} > flask.log 2>&1 &
                    sleep 5  # wait a bit for the app to start
                    if lsof -i:${APP_PORT}; then
                        echo "App started successfully on port ${APP_PORT}"
                    else
                        echo "App failed to start on port ${APP_PORT}"
                        exit 1
                    fi
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment succeeded"
        }
        failure {
            echo "❌ Deployment failed"
        }
    }
}

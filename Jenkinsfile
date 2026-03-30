pipeline {
    agent any

    environment {
        VENV_DIR = "${WORKSPACE}/venv"
        APP_PORT = "5000"
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
                #!/bin/bash
                set -e  # fail on real errors
                set -x  # print commands for debugging

                # Remove old virtualenv if exists
                rm -rf "${VENV_DIR}"

                # Create virtual environment
                python3 -m venv "${VENV_DIR}"

                # Activate virtualenv
                source "${VENV_DIR}/bin/activate"

                # Upgrade pip (ignore minor failures)
                pip install --upgrade pip || true

                # Install required Python packages
                pip install -r requirements.txt
                '''
            }
        }

        stage('Stop Old App') {
            steps {
                sh '''
                #!/bin/bash
                set +e  # don't fail if no process is running

                OLD_PID=$(lsof -t -i:${APP_PORT})
                if [ -n "$OLD_PID" ]; then
                    echo "Stopping old Flask process: $OLD_PID"
                    kill -9 $OLD_PID
                else
                    echo "No old process found on port ${APP_PORT}"
                fi
                '''
            }
        }

        stage('Start App') {
            steps {
                sh '''
                #!/bin/bash
                set -e
                set -x

                # Activate virtualenv
                source "${VENV_DIR}/bin/activate"

                # Start Flask app in background (logging output)
                nohup python3 app.py > flask.log 2>&1 &

                echo "Flask app started on port ${APP_PORT}. Logs: ${WORKSPACE}/flask.log"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment succeeded"
        }
        failure {
            echo "❌ Deployment failed. Check logs for details."
        }
    }
}

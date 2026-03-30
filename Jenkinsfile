pipeline {
    agent any

    environment {
        VENV_DIR = "${WORKSPACE}/venv"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Setup Environment') {
            steps {
                sh """
                # Clean previous virtualenv
                rm -rf ${VENV_DIR}
                python3 -m venv ${VENV_DIR}

                # Activate venv and upgrade pip
                . ${VENV_DIR}/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                """
            }
        }

        stage('Stop Old App') {
            steps {
                sh """
                OLD_PID=\$(lsof -t -i:5000)
                if [ ! -z "\$OLD_PID" ]; then
                    echo "Stopping old app with PID \$OLD_PID"
                    kill -9 \$OLD_PID
                else
                    echo "No old app running on port 5000"
                fi
                """
            }
        }

        stage('Start App') {
            steps {
                sh """
                # Activate virtual environment
                . ${VENV_DIR}/bin/activate

                # Start Flask app in background and log output
                nohup python app.py > flask.log 2>&1 &

                # Wait until port 5000 is open or timeout
                TIMEOUT=15
                while ! lsof -i:5000 >/dev/null 2>&1 && [ \$TIMEOUT -gt 0 ]; do
                    sleep 1
                    TIMEOUT=\$((TIMEOUT-1))
                done

                if lsof -i:5000 >/dev/null 2>&1; then
                    echo "✅ App started successfully on port 5000"
                else
                    echo "❌ App failed to start on port 5000"
                    echo "---- Flask log ----"
                    cat flask.log
                    exit 1
                fi
                """
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment succeeded"
        }
        failure {
            echo "❌ Deployment failed"
        }
    }
}

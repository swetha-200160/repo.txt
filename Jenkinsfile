
pipeline {
    agent any

    environment {
        BUILD_OUTPUT = "C:/Users/swethasuresh/target"
        VENV = "${env.WORKSPACE}\\venv"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    // replace with your repo url and credentialsId if needed
                    url: 'https://github.com/<your-username>/<your-repo>.git'
            }
        }

        stage('Setup venv & Install') {
            steps {
                bat """
                python -m venv "%VENV%"
                call "%VENV%\Scripts\activate"
                python -m pip install --upgrade pip setuptools wheel
                if exist requirements.txt (
                    pip install -r requirements.txt
                )
                pip freeze > "%WORKSPACE%\\venv-installed.txt"
                """
            }
        }

        stage('Test') {
            steps {
                bat """
                call "%VENV%\Scripts\activate"
                if exist tests (
                    pytest -q
                ) else (
                    echo No tests directory found
                )
                """
            }
        }

        stage('Build Package') {
            steps {
                bat """
                call "%VENV%\Scripts\activate"
                if exist dist rmdir /s /q dist || exit /b 0
                python -m pip wheel . -w dist
                python setup.py sdist bdist_wheel || echo "setuptools build finished"
                dir dist
                """
            }
        }

        stage('Copy Artifacts to Local') {
            steps {
                script {
                    bat """
                    if not exist "${env.BUILD_OUTPUT}" mkdir "${env.BUILD_OUTPUT}"
                    copy /Y "%WORKSPACE%\dist\*.*" "${env.BUILD_OUTPUT}\\">nul 2>&1 || echo "No artifacts to copy"
                    dir "${env.BUILD_OUTPUT}"
                    exit /b 0
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Build & copy succeeded'
            archiveArtifacts artifacts: 'dist/**', fingerprint: true
        }
        failure {
            echo 'Build failed - check console output'
        }
        always {
            echo 'Pipeline finished.'
        }
    }
}

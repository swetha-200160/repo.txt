pipeline {
  agent any

  environment {
    VENV = 'venv'
    REPORT_DIR = 'reports'
    DIST_DIR = 'dist'
  }

  stages {
    stage('Checkout') {
      steps {
        echo "Checking out repository..."
        checkout scm
      }
    }

    stage('Create venv & upgrade pip') {
      steps {
        script {
          if (isUnix()) {
            sh '''
              python3 -m venv ${VENV}
              ${VENV}/bin/python -m pip install --upgrade pip setuptools wheel
            '''
          } else {
            // Use a multiline single-quoted bat block. Use venv python directly — avoid "activate".
            bat '''
            python -m venv %VENV%
            "%VENV%\\Scripts\\python.exe" -m pip install --upgrade pip setuptools wheel
            '''
          }
        }
      }
    }

    stage('Install dependencies') {
      steps {
        script {
          if (isUnix()) {
            sh '''
              if [ -f requirements.txt ]; then
                ${VENV}/bin/python -m pip install -r requirements.txt
              else
                echo "No requirements.txt found; skipping pip install"
              fi
            '''
          } else {
            bat '''
            if exist requirements.txt (
              "%VENV%\\Scripts\\python.exe" -m pip install -r requirements.txt
            ) else (
              echo No requirements.txt found; skipping pip install
            )
            '''
          }
        }
      }
    }

    stage('Run tests (pytest)') {
      steps {
        script {
          if (isUnix()) {
            sh '''
              mkdir -p ${REPORT_DIR}
              ${VENV}/bin/python -m pip install -U pytest || true
              ${VENV}/bin/pytest --junitxml=${REPORT_DIR}/results.xml || true
            '''
          } else {
            bat '''
            if not exist %REPORT_DIR% mkdir %REPORT_DIR%
            "%VENV%\\Scripts\\python.exe" -m pip install -U pytest || exit /b 0
            "%VENV%\\Scripts\\pytest" --junitxml=%REPORT_DIR%\\results.xml || exit /b 0
            '''
          }
        }
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: "${REPORT_DIR}/*.xml"
        }
      }
    }

    stage('Build package (optional)') {
      steps {
        script {
          if (isUnix()) {
            sh '''
              if [ -f pyproject.toml ] || [ -f setup.py ]; then
                ${VENV}/bin/python -m pip install -U build || true
                rm -rf ${DIST_DIR}
                ${VENV}/bin/python -m build --outdir ${DIST_DIR} || true
              else
                echo "No pyproject.toml or setup.py found; skipping build"
              fi
            '''
            archiveArtifacts artifacts: "${DIST_DIR}/**", allowEmptyArchive: true
          } else {
            bat '''
            set BUILD_OK=0
            if exist pyproject.toml set BUILD_OK=1
            if exist setup.py set BUILD_OK=1
            if %BUILD_OK%==1 (
              "%VENV%\\Scripts\\python.exe" -m pip install -U build || exit /b 0
              if exist %DIST_DIR% rmdir /s /q %DIST_DIR%
              "%VENV%\\Scripts\\python.exe" -m build --outdir %DIST_DIR% || exit /b 0
            ) else (
              echo No pyproject.toml or setup.py found; skipping build
            )
            '''
            archiveArtifacts artifacts: "${DIST_DIR}\\\\**", allowEmptyArchive: true
          }
        }
      }
    }

    stage('Run script (example)') {
      steps {
        script {
          if (isUnix()) {
            sh '${VENV}/bin/python main.py'
          } else {
            bat '"%VENV%\\Scripts\\python.exe" main.py'
          }
        }
      }
    }
  } // stages

  post {
    success { echo "Pipeline succeeded" }
    unstable { echo "Pipeline finished with unstable results" }
    failure { echo "Pipeline failed" }
    always { cleanWs() }
  }
}

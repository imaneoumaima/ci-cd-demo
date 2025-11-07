pipeline {
  agent { label 'built-in' }

  environment {
    PYTHON = 'python3'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Setup venv') {
      steps {
        sh '''
          ${PYTHON} -m venv .venv
          . .venv/bin/activate
          pip install --upgrade pip
          pip install -r requirements.txt
        '''
      }
    }

    stage('Unit tests') {
      steps {
        sh '''
          . .venv/bin/activate
          pytest -q --junitxml=reports/junit.xml
        '''
      }
      post {
        always {
          junit 'reports/junit.xml'
        }
      }
    }

    stage('Build Docker image') {
      steps {
        sh '''
          docker build -t ci-cd-demo:latest .
        '''
      }
    }

    stage('Deploy to Prod (local)') {
      when { branch 'main' }
      steps {
        sh '''
          (docker rm -f app-prod || true)
          docker run -d --name app-prod -p 5000:5000 ci-cd-demo:latest
        '''
      }
    }
  }

  post {
    success { echo 'Pipeline OK ✅' }
    always  { cleanWs() }
  }
}

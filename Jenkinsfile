pipeline {
  agent any

  parameters {
    booleanParam(name: 'DEPLOY_LOCAL', defaultValue: true, description: 'Deploy to local Docker?')
    string(name: 'APP_PORT', defaultValue: '8081', description: 'Application port')
    string(name: 'IMAGE', defaultValue: 'sit753-my-app', description: 'Docker image name')
    string(name: 'VERSION', defaultValue: 'latest', description: 'Docker image version')
  }

  tools {
    jdk 'java25'
    maven 'maven3'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        bat 'mvn -B -DskipTests package'
      }
    }

    stage('Test') {
      steps {
        bat 'mvn -B test'
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
        }
      }
    }

    stage('Code Quality') {
      steps {
        bat 'mvn -B verify'
      }
    }

    stage('Security') {
      steps {
        bat '''
          echo Running Trivy security scan...
          trivy fs --exit-code 0 --no-progress . || echo Security scan completed with warning
        '''
      }
    }

    stage('Deploy') {
      when {
        expression { return params.DEPLOY_LOCAL }
      }
      steps {
        bat '''
          docker build -t %IMAGE%:%VERSION% -f Dockerfile .
          docker stop sit753-my-app || exit 0
          docker rm sit753-my-app || exit 0
          docker run -d --name sit753-my-app -p %APP_PORT%:8081 %IMAGE%:%VERSION%
        '''
      }
    }

    stage('Release') {
      steps {
        bat '''
          echo Release created for application: %IMAGE%
          echo Release version: %VERSION%
          echo This build is marked as the stable release after successful pipeline stages.
        '''
      }
    }

    stage('Monitoring') {
      steps {
        bat '''
          echo Waiting for application to start...
          timeout /t 10
          echo Checking health endpoint...
          curl http://localhost:%APP_PORT%/actuator/health
        '''
      }
    }
  }

  post {
    success {
      echo 'Pipeline completed successfully.'
    }
    failure {
      echo 'Pipeline failed. Please check console output.'
    }
  }
}
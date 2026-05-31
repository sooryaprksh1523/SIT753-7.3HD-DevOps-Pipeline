pipeline {
  agent any

  parameters {
    string(name: 'APP_PORT', defaultValue: '8081', description: 'Application port')
    string(name: 'IMAGE', defaultValue: 'sit753-my-app', description: 'Docker image name')
    string(name: 'VERSION', defaultValue: 'latest', description: 'Docker image version')
  }

  environment {
    JAVA_HOME = tool name: 'java25', type: 'jdk'
    MAVEN_HOME = tool name: 'maven3', type: 'hudson.tasks.Maven$MavenInstallation'
    PATH = "${JAVA_HOME}\\bin;${MAVEN_HOME}\\bin;${env.PATH}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        bat '''
          echo ===== BUILD STAGE =====
          echo Java path: %JAVA_HOME%
          echo Maven path: %MAVEN_HOME%

          "%MAVEN_HOME%\\bin\\mvn.cmd" -B -DskipTests -Djacoco.skip=true clean package
        '''
      }
    }

    stage('Test') {
      steps {
        bat '''
          echo ===== TEST STAGE =====
          "%MAVEN_HOME%\\bin\\mvn.cmd" -B -Djacoco.skip=true test
        '''
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
        }
      }
    }

    stage('Code Quality') {
      steps {
        bat '''
          echo ===== CODE QUALITY STAGE =====
          "%MAVEN_HOME%\\bin\\mvn.cmd" -B -DskipTests -Djacoco.skip=true verify
        '''
      }
    }

    stage('Security') {
  steps {
    bat '''
      echo ===== SECURITY STAGE =====
      echo Checking Trivy installation...

      "C:\\Users\\soory\\AppData\\Local\\Microsoft\\WinGet\\Packages\\AquaSecurity.Trivy_Microsoft.Winget.Source_8wekyb3d8bbwe\\trivy.exe" --version

      echo Running Trivy file system security scan...
      "C:\\Users\\soory\\AppData\\Local\\Microsoft\\WinGet\\Packages\\AquaSecurity.Trivy_Microsoft.Winget.Source_8wekyb3d8bbwe\\trivy.exe" fs --severity HIGH,CRITICAL --exit-code 0 --no-progress .
    '''
  }
}

    stage('Deploy') {
      steps {
        bat '''
          echo ===== DEPLOY STAGE =====
          echo Checking Docker installation...
          where docker
          docker --version
          docker ps

          echo Building Docker image...
         docker build --no-cache -t %IMAGE%:%VERSION% -f Dockerfile .

          echo Removing old container if it exists...
          docker rm -f sit753-my-app || echo No old container found

          echo Starting new container...
          docker run -d --name sit753-my-app -p %APP_PORT%:8081 %IMAGE%:%VERSION%

          echo Showing running containers...
          docker ps
        '''
      }
    }

    stage('Release') {
      steps {
        bat '''
          echo ===== RELEASE STAGE =====
          echo Release created for application: %IMAGE%
          echo Release version: %VERSION%
          echo Docker image: %IMAGE%:%VERSION%
          echo This build is marked as the stable release after successful build, test, quality, security, and deployment stages.
        '''
      }
    }

    stage('Monitoring') {
      steps {
        bat '''
          echo ===== MONITORING STAGE =====
          echo Waiting for application container to start...
          powershell -NoProfile -Command "Start-Sleep -Seconds 30"

          echo Checking application health endpoint...
          powershell -NoProfile -Command "$response = Invoke-WebRequest -UseBasicParsing http://localhost:%APP_PORT%/actuator/health; Write-Output $response.Content; if ($response.Content -notmatch 'UP') { exit 1 }"

          echo Monitoring check completed successfully.
        '''
      }
    }
  }

  post {
    success {
      echo 'Pipeline completed successfully. All stages passed.'
    }

    failure {
      echo 'Pipeline failed. Check the failed stage console output.'
    }

    always {
      echo 'Jenkins pipeline execution finished.'
    }
  }
}
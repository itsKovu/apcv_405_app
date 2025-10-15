pipeline {
  agent any
  options { timestamps() }

  environment {
    IMAGE_NAME     = "itskovu/apcv_405_app"
    CONTAINER_NAME = "apcv_app"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: 'https://github.com/itsKovu/apcv_405_app.git']]
        ])
      }
    }

    stage('Run Tests') {
      steps {
        bat 'dotnet test apcv_405_app.sln --nologo'
      }
    }

    stage('Build Docker Image') {
      steps {
        bat "docker build -t %IMAGE_NAME%:latest -t %IMAGE_NAME%:%BUILD_NUMBER% ."
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials',
          usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          bat 'echo %DH_PASS% | docker login -u %DH_USER% --password-stdin'
          bat 'docker push %IMAGE_NAME%:latest'
          bat 'docker push %IMAGE_NAME%:%BUILD_NUMBER%'
        }
      }
    }

    stage('Deploy Locally') {
      steps {
        bat 'docker rm -f %CONTAINER_NAME% 2>nul || ver > nul'
        bat 'docker run -d --name %CONTAINER_NAME% -p 9090:8080 %IMAGE_NAME%:latest'
      }
    }

    stage('Health Check') {
      steps {
        bat 'powershell -Command "$r=Invoke-WebRequest http://localhost:9090/health -UseBasicParsing; if($r.StatusCode -ne 200){Write-Error \\"Health check failed\\"} else {Write-Host \\"Health OK\\"}"'
      }
    }
  }

  post { always { echo 'Pipeline finished.' } }
}

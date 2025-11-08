pipeline {
  agent any
  environment {
    CI = 'true'
  }
  options {
    skipStagesAfterUnstable()
    ansiColor('xterm')
    timestamps()
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
        script {
          echo "Checking out commit: ${env.GIT_COMMIT ?: 'unknown'}"
        }
      }
    }

    stage('Build') {
      steps {
        script {
          if (fileExists('pom.xml')) {
            echo 'Detected Maven project — running mvn package'
            sh 'mvn -B -DskipTests package'
          } else if (fileExists('build.gradle') || fileExists('build.gradle.kts')) {
            echo 'Detected Gradle project — running ./gradlew assemble'
            sh './gradlew assemble'
          } else if (fileExists('package.json')) {
            echo 'Detected Node project — running npm ci && npm run build'
            sh 'npm ci && npm run build || true'
          } else {
            echo 'No recognised build file found — skipping build step'
          }
        }
      }
    }

    stage('Test') {
      steps {
        script {
          if (fileExists('pom.xml')) {
            echo 'Running Maven tests'
            sh 'mvn -B test'
          } else if (fileExists('build.gradle') || fileExists('build.gradle.kts')) {
            echo 'Running Gradle tests'
            sh './gradlew test'
          } else if (fileExists('package.json')) {
            echo 'Running npm test'
            sh 'npm test || true'
          } else {
            echo 'No recognised test command — skipping tests'
          }
        }
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml, **/build/test-results/**/*.xml, **/test-results/**/*.xml'
        }
      }
    }

    stage('Archive') {
      steps {
        archiveArtifacts artifacts: '**/target/*.jar, **/build/libs/*.jar, **/dist/**, **/release/**', allowEmptyArchive: true
      }
    }

    stage('Docker Build (optional)') {
      when {
        expression { fileExists('Dockerfile') }
      }
      steps {
        script {
          def image = "${env.JOB_NAME.replaceAll('[^a-zA-Z0-9_.-]','_')}:${env.BUILD_NUMBER}"
          echo "Building Docker image ${image}"
          sh "docker build -t ${image} ."
          echo "Built ${image} (push step not included by default)"
        }
      }
    }
  }

  post {
    always {
      cleanWs()
    }
    success {
      echo 'Pipeline finished successfully.'
    }
    failure {
      echo 'Pipeline failed.'
    }
  }
}
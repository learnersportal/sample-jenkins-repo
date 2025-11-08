pipeline {
  agent any

  stages {
    stage('Fetch Code') {
      steps {
        echo 'Fetching source code...'
        checkout scm
      }
    }

    stage('Install Apache') {
      steps {
        echo 'Installing Apache...'
        sh '''
          if ! command -v apache2 >/dev/null 2>&1; then
            sudo apt-get update -y
            sudo apt-get install -y apache2
          else
            echo "Apache already installed"
          fi
        '''
      }
    }

    stage('Deploy') {
      steps {
        echo 'Deploying application to /var/www/html'
        sh '''
          sudo rsync -av --delete ./ /var/www/html/
          sudo chown -R www-data:www-data /var/www/html
          sudo systemctl restart apache2 || true
        '''
      }
    }
  }

  post {
    always {
      echo 'Pipeline finished'
      cleanWs()
    }
  }
}
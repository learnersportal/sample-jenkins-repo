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
  stage('Deploy Code') {
         steps {
           echo 'Deploying application to /var/www/html'
             script {
                  def userInput = input(
                  id: 'DeployConfirm', message: 'Do you want to deploy?', 
                  parameters: [choice(name: 'Deploy', choices: ['Yes', 'No'], description: 'Confirm deployment')])
                  if (userInput == 'No') {
                      error('Deployment aborted by user.')
                  }else{
                      sh 'sudo cp * /var/www/html/'
                  }
             } 
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

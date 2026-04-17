pipeline {
  agent any

  stages {

    stage("Initial cleanup") {
      steps {
        deleteDir()
      }
    }
    stage('Chcekout SCM') {
      steps { 
          git branch: 'main' , url:
            'https://github.com/StegTechHub/php-todo.git'
      }
    }
  
    stage('Prepare Dependencies') {
      steps {
        script {
          docker.image('php:7.4-cli').inside('-u root') {
            sh '''
              apt-get update
              apt-get install -y unzip curl git

              curl -sS https://getcomposer.org/installer | php
              mv composer.phar /usr/local/bin/composer

              echo "APP_ENV=testing" > .env
              echo "APP_KEY=" >> .env
              echo "APP_DEBUG=true" >> .env
              echo "DB_CONNECTION=sqlite" >> .env
              echo "DB_DATABASE=database/database.sqlite" >> .env
              
              mkdir -p database
              touch database/database.sqlite

              composer install
              php artisan key:generate
              php artisan migrate --force
              php artisan db:seed --force
            '''
          }
        }
      }
    }

    stage('Execute Unit Tests') {
      steps {
        script {
          docker.image('php:7.4-cli').inside {
            sh './vendor/bin/phpunit'
          }
        }
      }
    }
  }
}


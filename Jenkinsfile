pipeline {
  agent any

  stages {

    stage("Initial cleanup") {
      steps {
        deleteDir()
      }
    }

    stage('Checkout SCM') {
      steps {
        git branch: 'main', url: 'https://github.com/StegTechHub/php-todo.git'
      }
    }

    stage('Prepare Dependencies') {
      steps {
        sh '''
        echo "APP_ENV=testing" > .env
        echo "APP_KEY=" >> .env
        echo "APP_DEBUG=true" >> .env
        echo "DB_CONNECTION=sqlite" >> .env
        echo "DB_DATABASE=database/database.sqlite" >> .env

        touch database/database.sqlite
        composer install
        php artisan migrate
        php artisan db:seed
        php artisan key:generate
        '''
      }
    }

    stage('Execute Unit Tests') {
      steps {
        sh './vendor/bin/phpunit'
      }
    }
  }
}

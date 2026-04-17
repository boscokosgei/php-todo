pipeline {
  agent {
    docker {
      image 'php:7.4-cli'
    }
  }
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
        apt-get update
        apt-get install -y unzip curl git

        curl -sS https://getcomposer.org/installer | php
        mv composer.phar /usr/local/bin/composer
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

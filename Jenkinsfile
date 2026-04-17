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
        script {
          docker.image('php:7.4-cli').inside('-u root') {
            sh '''
              apt-get update
              apt-get install -y unzip curl git zip

              curl -sS https://getcomposer.org/installer | php
              mv composer.phar /usr/local/bin/composer

              echo "APP_ENV=testing" > .env
              echo "APP_KEY=" >> .env
              echo "APP_DEBUG=true" >> .env
              echo "DB_CONNECTION=sqlite" >> .env
              echo "DB_DATABASE=database/database.sqlite" >> .env

              mkdir -p database
              touch database/database.sqlite

              mkdir -p bootstrap/cache
              chmod -R 777 bootstrap/cache

              mkdir -p storage/framework/{sessions,views,cache}

              chmod -R 777 database storage

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

    stage('Code Analysis') {
      steps {
        script {
          docker.image('php:7.4-cli').inside('-u root') {
            sh '''
              curl -L https://phar.phpunit.de/phploc.phar -o phploc.phar
              chmod +x phploc.phar

              mkdir -p build/logs
              php phploc.phar app/ --log-csv build/logs/phploc.csv
            '''
          }
        }
      }
    }

    stage('Package Artifact') {
      steps {
        sh 'zip -qr php-todo.zip .'
      }
    }

    stage('Upload Artifact to Artifactory') {
      steps {
        script {
          def server = Artifactory.server 'artifactory-server'
          def uploadSpec = """ {
              "files":[
                {
                  "pattern": "php-todo.zip",
                  "target": "libs-release-local/php-todo/",
                  "props": "type=zip;status=ready"
                }
              ]
          }"""
          server.upload spec: uploadSpec
        }
      }
    }
  }
}


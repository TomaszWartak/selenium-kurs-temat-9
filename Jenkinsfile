pipeline {
    agent any
    tools {
            maven "Maven" // nazwa zdefiniowana w konfiguracji Jenkins
            //docker "Docker"
    }
    stages {
        stage('Build test code') {
            steps {
                sh 'mvn clean install -DskipTests' // Budowanie testów
            }
        }
        stage('Run selenium grid') {
            steps {
                sh 'docker-compose up -d' // Uruchiomienie Docker Selenium
            }
        }
        stage('Execute test') {
            steps {
                sh 'mvn test' // Uruchomienie testów
                sh 'docker compose down' // Wyłączenie Docker Selenium, wyłączenie kontenerów
            }
        }
    }
    post {
        always {
            script { // Wygenerowanie raportu allurowego
                allure([
                        includeProperties: false,
                        jdk              : '',
                        properties       : [],
                        reportBuildPolicy: 'ALWAYS',
                        results          : [[path: 'target/allure-results']]
                ])
            }
        }
    }
}
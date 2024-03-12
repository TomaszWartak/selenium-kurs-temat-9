pipeline {
    agent any
    tools {
            maven "Maven" // nazwa zdefiniowana w konfiguracji Jenkins
            docker "Docker"
            'org.jenkinsci.plugins.docker.commons.tools.DockerTool' 'docker'
    }
    stages {
        stage('Build test code') {
            steps {
                sh 'mvn clean install -DskipTests' // Budowanie testów
            }
        }
        stage('Run selenium grid') {
            steps {
                sh 'docker compose up -d' // Uruchomienie Docker Selenium
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
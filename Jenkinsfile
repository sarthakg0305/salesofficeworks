pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/sarthakg0305/salesofficeworks.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        cd management
                        ./mvnw clean verify sonar:sonar \
                        -Dsonar.projectKey=salesofficeworks \
                        -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Backend') {
            steps {
                sh 'cd management && ./mvnw clean package -DskipTests'
            }
        }

        stage('Docker Build Backend') {
            steps {
                sh 'cd management && docker build -t sales-backend .'
            }
        }

        stage('Run Backend') {
            steps {
                sh 'docker stop backend || true'
                sh 'docker rm backend || true'
                sh 'docker run -d -p 8081:8080 --name backend sales-backend'
            }
        }

        stage('Build Frontend') {
            steps {
                sh 'cd salesmanagement1 && npm install && npm run build'
            }
        }

        stage('Docker Build Frontend') {
            steps {
                sh 'cd salesmanagement1 && docker build -t sales-frontend .'
            }
        }

        stage('Run Frontend') {
            steps {
                sh 'docker stop frontend || true'
                sh 'docker rm frontend || true'
                sh 'docker run -d -p 80:80 --name frontend sales-frontend'
            }
        }
    }
}

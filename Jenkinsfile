pipeline {
    agent any

    environment {
        scannerHome = tool 'SonarQube'
        SONARQUBE_TOKEN = credentials('SONARQUBE_TOKEN')
        DOCKER_HUB_PASSWORD = credentials('12dd7f66-4f99-489c-8feb-27ce1005bf9e')
    }

    stages {
        stage('Clear running apps') {
            steps {
                sh 'docker rm -f devops_flask_app || true'
            }
        }

        stage('Sonarqube analysis frontend') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.login=${SONARQUBE_TOKEN}"
                }
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t devops_flask_app:${BUILD_NUMBER} -t devops_flask_app:latest ."
            }
        }
        stage('Run app') {
            steps {
                sh "docker run -d -p 0.0.0.0:5555:5555 --net=environment_docker_network --name devops_flask_app -t devops_flask_app:${BUILD_NUMBER}"
            }
        }
        stage('Selenium tests') {
            steps {
                dir('tests/') {
                    sh 'pip3 install -r requirements.txt'
                    sh 'python3 test_app.py'
                }
            }
        }
        stage('Upload Docker Image to Docker Hub') {
            steps {
                sh "docker login -u wdemski -p ${DOCKER_HUB_PASSWORD}"
                sh "docker tag devops_flask_app:${BUILD_NUMBER} wdemski/devops_flask_app:${BUILD_NUMBER}"
                sh 'docker tag devops_flask_app:latest wdemski/devops_flask_app:latest'
                sh "docker push wdemski/devops_flask_app:${BUILD_NUMBER}"
                sh 'docker push wdemski/devops_flask_app:latest'
            }
        }
    }
}

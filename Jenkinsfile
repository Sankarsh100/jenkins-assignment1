pipeline {
    agent { label 'maven-agent' }

    environment {
        APP_SERVER = '54.237.244.138'
        APP_USER = 'ubuntu'
        DEPLOY_PATH = '/home/ubuntu/app.jar'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('Security Scan') {
            steps {
                sh 'mvn org.owasp:dependency-check-maven:check || true'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }

        stage('Deploy to Application Server') {
            when {
                branch 'main'
            }
            steps {
                sshagent(credentials: ['app-server-ssh']) {
                    sh '''
                        JAR_FILE=$(ls target/*.jar | head -n 1)
                        scp -o StrictHostKeyChecking=no "$JAR_FILE" ${APP_USER}@${APP_SERVER}:${DEPLOY_PATH}
                        ssh -o StrictHostKeyChecking=no ${APP_USER}@${APP_SERVER} '
                            pkill -f "java -jar" || true
                            nohup java -jar /home/ubuntu/app.jar > /home/ubuntu/app.log 2>&1 &
                        '
                    '''
                }
            }
        }
    }
}
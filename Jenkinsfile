pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://32.236.85.107:9000'
        PROJECT_DIR = 'test_django_pro'
        IMAGE_NAME = 'django-app'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Environment') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    . venv/bin/activate
                    pip install -r ${PROJECT_DIR}/requirements.txt
                    pip install pytest pytest-django pytest-cov
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    export PYTHONPATH=$WORKSPACE

                    cd ${PROJECT_DIR}

                    python manage.py makemigrations --check --dry-run
                    python manage.py migrate

                    pytest \
                      --ds=test_django_pro.settings \
                      --junitxml=report.xml \
                      --cov=. \
                      --cov-report=xml || true
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh '''
                    docker run --rm \
                      -e SONAR_HOST_URL=$SONAR_HOST_URL \
                      -v $WORKSPACE:/usr/src \
                      sonarsource/sonar-scanner-cli \
                      -Dsonar.projectKey=django-app \
                      -Dsonar.sources=/usr/src \
                      -Dsonar.host.url=$SONAR_HOST_URL \
                      -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME ${PROJECT_DIR}
                '''
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh '''
                    docker stop django-staging || true
                    docker rm django-staging || true
                    docker run -d -p 8001:8000 --name django-staging $IMAGE_NAME
                '''
            }
        }

        stage('Approval for Production') {
            steps {
                input message: 'Deploy to Production?'
            }
        }

        stage('Deploy to Production') {
            steps {
                sh '''
                    docker stop django-prod || true
                    docker rm django-prod || true
                    docker run -d -p 8000:8000 --name django-prod $IMAGE_NAME
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS ✔"
        }

        failure {
            echo "Pipeline FAILED ❌"
        }

        always {
            cleanWs()
        }
    }
}

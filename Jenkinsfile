pipeline {
    agent any

    environment {
        IMAGE_NAME = "django-app"
        STAGING_CONTAINER = "django-staging"
        PROD_CONTAINER = "django-prod"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/ananpabbas/django-ci-cd-project.git'
            }
        }

        stage('Create Virtual Environment') {
            steps {
                sh '''
                python3 -m venv venv
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                . venv/bin/activate
                python manage.py test
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME .
                '''
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh '''
                docker stop $STAGING_CONTAINER || true
                docker rm $STAGING_CONTAINER || true
                docker run -d -p 8001:8000 --name $STAGING_CONTAINER $IMAGE_NAME
                '''
            }
        }

        stage('Approval for Production') {
            steps {
                input message: "Deploy to Production?"
            }
        }

        stage('Deploy to Production') {
            steps {
                sh '''
                docker stop $PROD_CONTAINER || true
                docker rm $PROD_CONTAINER || true
                docker run -d -p 8000:8000 --name $PROD_CONTAINER $IMAGE_NAME
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully 🎉"
        }

        failure {
            echo "Pipeline failed ❌ Check logs"
        }
    }
}

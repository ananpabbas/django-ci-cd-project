pipeline {
    agent any

    environment {
        IMAGE_NAME = "django-app"
        STAGING_CONTAINER = "django-staging"
        PROD_CONTAINER = "django-prod"
        WORKSPACE_DIR = "${env.WORKSPACE}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/ananpabbas/django-ci-cd-project.git'
            }
        }

        stage('Setup Environment') {
            steps {
                sh '''
                cd $WORKSPACE_DIR
                python3 -m venv venv
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                cd $WORKSPACE_DIR
                export PYTHONPATH=$WORKSPACE_DIR
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                cd $WORKSPACE_DIR
                export PYTHONPATH=$WORKSPACE_DIR
                . venv/bin/activate
                python manage.py test
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                cd $WORKSPACE_DIR
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
            echo "✅ Pipeline SUCCESS"
        }

        failure {
            echo "❌ Pipeline FAILED - check logs"
        }
    }
}

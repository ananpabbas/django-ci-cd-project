pipeline {
    agent any

    stages {
        stage('GitHub Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ananpabbas/django-ci-cd-project.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Build Successful'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'SonarQube Analysis Completed'
            }
        }
    }
}

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                git credentialsId: 'github', url: 'https://github.com/julianomontini/sample-app.git'
                sh label: 'Install npm dependencies', script: 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh label: 'Run unit tests', script: 'npm test'
            }
        }
        stage('Integration Tests'){
            when {
                branch 'master' 
            }
            steps {
                sh label: 'Run integration tests', script: 'echo \"Running integration tests\"'
            }
        }
        stage('Deploy Development') {
            when {
                branch 'development' 
            }
            steps {
                withCredentials([file(credentialsId: 'gcp_service_account_staging', variable: 'SA_KEY')]) {
                    configFileProvider([configFile(fileId: 'app_config_testing', targetLocation: 'app.yaml')]) {
                        sh label: 'Activate service account', script: 'gcloud auth activate-service-account --key-file=$SA_KEY'
                        sh label: 'Deploy APP', script: 'gcloud app deploy'   
                    }
                }
            }
        }
        stage('Deploy Production') {
            when {
                branch 'master' 
            }
            steps {
                milestone(1)
                input message: "Do you REALLY want to deploy to production?"
                milestone(2)
                withCredentials([file(credentialsId: 'gcp_service_account_production', variable: 'SA_KEY')]) {
                    configFileProvider([configFile(fileId: 'app_config_production', targetLocation: 'app.yaml')]) {
                        sh label: 'Activate service account', script: 'gcloud auth activate-service-account --key-file=$SA_KEY'
                        sh label: 'Deploy APP', script: 'gcloud app deploy'   
                    }
                }
            }
        }
    }
}

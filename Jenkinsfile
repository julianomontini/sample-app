pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
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
                        lock('GCP_CLI'){
                          sh label: 'Activate service account', script: 'gcloud auth activate-service-account --key-file=$SA_KEY --project jenkins-testing-282513'
                          sh label: 'Deploy APP', script: 'gcloud app deploy'   
                        }
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
                withCredentials([file(credentialsId: 'gcp_service_account_production', variable: 'SA_KEY_PROD')]) {
                    configFileProvider([configFile(fileId: 'app_config_production', targetLocation: 'app.yaml')]) {
                        lock('GCP_CLI'){
                          sh label: 'Activate service account', script: 'gcloud auth activate-service-account --key-file=$SA_KEY_PROD --project jenkins-production-282520'
                          sh label: 'Deploy APP', script: 'gcloud app deploy'   
                        }
                    }
                }
            }
        }
    }
}

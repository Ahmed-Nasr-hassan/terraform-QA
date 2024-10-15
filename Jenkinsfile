pipeline {
    agent {label 'master'}

    stages {
        stage('CI') {
            steps {
                git url: 'https://github.com/Ahmed-Nasr-hassan/terraform-QA', branch: 'main'
                sh '''
                  terraform init
                  terraform plan
                '''
            }
        }
        
    }
}

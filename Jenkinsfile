pipeline {
    agent {label 'master'}

    stages {
        stage('CI') {
            steps {
                git url: 'https://github.com/Ahmed-Nasr-hassan/terraform-QA', branch: 'main'
                sh '''
                  terraform init
                  terraform apply -auto-approve
                '''
            }
            
            post {
                success {
                    slackSend (color: '#34ebc9', message: 'Infrastructure building Success ya Nasr, check it from ' + BUILD_URL)
                }
                
                failure {
                    slackSend (color: '#eb349e', message: 'Infrastructure building Failed ya Nasr, check it from ' + BUILD_URL)
                }
            }
        }
        
    }
}

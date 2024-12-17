pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION='us-east-1'
        AWS_ECS_CLUSTER='jenkins'
        AWS_ECS_SERVICE='jenkins-service'
        AWS_ECS_TD='jenkins-prod'
    }
    
    stages {
        stage('Deploy to AWS ECS') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.22.18'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'admin-general-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        yum install jq -y
                        LATEST_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition.json | jq '.taskDefinition.revision')
                        echo $LATEST_REVISION
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TD:$LATEST_REVISION
                        aws ecs wait services-table --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                    '''
                }
            }
        }
    }
}
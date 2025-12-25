pipeline {

    agent {
        label "AGENT-1"
    }

    environment{
        appVersion=""
        region='us-east-1'
        account="759713897143"
        project="roboshop"
        component="catalogue"
    }
    options{
        timeout(time: 30, unit: "MINUTES")
        disableConcurrentBuilds()
    }

    parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        choice(name:'deply_to', choices:['dev', 'qa', 'prod'], description:'Pick something')
    }
    
    
    stages {

        stage('Deploy'){
            steps{
                script{
                    withAWS(region:'us-east-1', credentials: 'aws-creds') {            
                    sh """
                        aws eks-update kubeconfig --region $region --name "$project-${params.deploy_to}"
                        kubectl get nodes
                        sed -i 's/IMAGE_VERSION/${params.deploy_to}/g' values-${param.deploy_to}.yaml
                        helm upgrade --install $component -f values-${param.deploy_to}.yaml -n $project .
                    """
                    }
                }
            }
        }
    }
    stage("Check Status"){
        steps{
            script{
                def deploymentStatus=sh(returnStdout: true, script: "kubectl rollout status deployment/catalogue --request-timeout=30s -n $project || echo FAILED").trim()
                if deploymentStatus.contains("Successfully rolledout"){
                    echo "deployment is success"
                }
                else{
                    sh """
                        helm rollback $component -n $project
                        sleep 20
                    """
                    def rollbackStatus=sh(returnStdout: true, script: "kubectl rollout status deployment/catalogue  --request-timeout=30s -n $project || echo FAILED").trim()
                    if rollbackStatus.contains("successfully rolledout"){
                        error "deployment is failure, rollback is success"
                    }
                    else{
                        error "Deployment is failure rollback is failure, application is not running"                       
    
                    }
                }

            }
        }
    }
    post{
        always{
            echo "I will always say hello again"
            deleteDir()
        }
        success{
            echo "Hello Success..."
        }
        failure{
            echo "Hello Failure..."
        }

    }
}
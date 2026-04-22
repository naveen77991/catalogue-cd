pipeline {
    agent {
        label 'catalogue'
    }
    environment {
        REGION = "us-west-1"
        ACC_ID = "439481669447"
        COMPONENT = "catalogue"
        CLUSTER = "catalogue-cluster1"
        NAMESPACE = "roboshop"
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    parameters {
        string(name: 'appVersion', description: 'Image version to deploy')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the Environment')
    }
    stages {
        stage('Configure kubectl') {
            steps {
                script {
                    sh """
                        aws eks update-kubeconfig --region ${REGION} --name ${CLUSTER}
                        kubectl get nodes
                    """
                }
            }
        }
        stage('Helm Deploy') {
            steps {
                script {
                    sh """
                        kubectl create namespace ${NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -
                        helm upgrade --install ${COMPONENT} \
                            -f values-${params.deploy_to}.yaml \
                            --set image.tag=${params.appVersion} \
                            --set image.repository=${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${COMPONENT} \
                            -n ${NAMESPACE} .
                    """
                }
            }
        }
        stage('Check Rollout') {
            steps {
                script {
                    sh """
                        kubectl rollout status deployment/${COMPONENT} \
                            --timeout=120s \
                            -n ${NAMESPACE}
                    """
                }
            }
        }
    }
    post {
        always {
            echo 'CD Pipeline finished!'
            deleteDir()
        }
        success {
            echo 'Deploy Success!'
        }
        failure {
            echo 'Deploy Failed!'
        }
    }
}

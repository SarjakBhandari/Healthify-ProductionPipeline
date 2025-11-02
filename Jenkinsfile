pipeline {
    agent { label 'ProductionENV' }

    environment {
        INVENTORY     = "hosts"
        PLAYBOOK      = "deploy.yml"
        MONITOR_PLAY  = "deploy_monitoring.yml"
        ANSIBLE_DIR   = "Healthify-ProductionPipeline"
    }

    stages {
        stage('Prepare Workspace') {
            steps {
                deleteDir()
                sh 'git clone https://github.com/SarjakBhandari/Healthify-ProductionPipeline'
            }
        }

        stage('Deploy Healthify Stack') {
            steps {
                dir("${ANSIBLE_DIR}") {
                    sh '''
                        set -e
                        echo ">>> Running Ansible Playbook for Healthify Deployment..."
                        ansible-playbook --syntax-check -i ${INVENTORY} ${PLAYBOOK}
                        ansible-playbook -i ${INVENTORY} ${PLAYBOOK}
                    '''
                }
            }
        }

        stage('Confirm Monitoring Deployment') {
            steps {
                script {
                    def userInput = input(
                        id: 'monitorConfirm', message: 'Do you want to deploy the monitoring stack?', parameters: [
                            booleanParam(defaultValue: false, description: 'Deploy monitor.yml?', name: 'DEPLOY_MONITORING')
                        ]
                    )
                    env.DEPLOY_MONITORING = userInput.toString()
                }
            }
        }

        stage('Deploy Monitoring Stack') {
            when {
                expression { env.DEPLOY_MONITORING == 'true' }
            }
            steps {
                dir("${ANSIBLE_DIR}") {
                    sh '''
                        set -e
                        echo ">>> Running Ansible Playbook for Monitoring Deployment..."
                        ansible-playbook --syntax-check -i ${INVENTORY} ${MONITOR_PLAY}
                        ansible-playbook -i ${INVENTORY} ${MONITOR_PLAY}
                    '''
                }
            }
        }
    }

    post {
        success {
            mail to: 'sarjakytdfiles@gmail.com',
                 subject: 'HEALTHIFY + MONITORING DEPLOYMENT SUCCESS',
                 body: """Deployment succeeded

App: http://192.168.50.4:5173  
API: http://192.168.50.4:5050  
Prometheus: http://192.168.50.4:9090  
Grafana: http://192.168.50.4:3000 (admin/admin)

Check services with:
docker service ls

Build URL: ${BUILD_URL}"""
        }
        failure {
            mail to: 'sarjakytdfiles@gmail.com',
                 subject: 'HEALTHIFY + MONITORING DEPLOYMENT FAILURE',
                 body: """Deployment failed 
Check Jenkins logs: ${BUILD_URL}"""
        }
    }
}

pipeline {
    agent { label 'ProductionENV' }

    environment {
        INVENTORY   = "hosts"
        PLAYBOOK    = "deploy.yml"
        ANSIBLE_DIR = "Healthify-ProductionPipeline"
    }

    stages {
        stage('Prepare Workspace') {
            steps {
                deleteDir()
                sh 'git clone https://github.com/SarjakBhandari/Healthify-ProductionPipeline'
            }
        }

    stage('Deploy Healthify & Monitoring Stack') {
        steps {
            dir("${ANSIBLE_DIR}") {
                sh '''
                    set -e
                    echo ">>> Running Ansible Playbook for Swarm Deployment..."
                    ansible-playbook --syntax-check -i ${INVENTORY} ${PLAYBOOK}
                    ansible-playbook -i ${INVENTORY} ${PLAYBOOK}
                '''
            }
        }
    }
     stage('Confirm Monitoring Deployment') {
        steps {
            script {
                def deployMonitoring = input(
                    id: 'monitoringConfirm',
                    message: 'Do you want to deploy the Prometheus & Grafana monitoring stack?',
                    parameters: [
                        choice(name: 'Deploy', choices: ['Yes', 'No'], description: 'Select Yes to proceed with monitoring deployment')
                    ]
                )

                if (deployMonitoring == 'No') {
                    env.SKIP_MONITORING = 'true'
                } else {
                    env.SKIP_MONITORING = 'false'
                }
            }
        }
    }

    stage('Deploy Monitoring Stack') {
        steps {
            dir("${ANSIBLE_DIR}") {
                sh '''
                    set -e
    
                    if [ "${SKIP_MONITORING}" = "false" ]; then
                        echo ">>> Deploying Monitoring Stack..."
                        ansible-playbook --syntax-check -i ${INVENTORY} monitor.yml
                        ansible-playbook -i ${INVENTORY} monitor.yml
                    else
                        echo ">>> Skipping Monitoring Stack Deployment."
                    fi
                '''
            }
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

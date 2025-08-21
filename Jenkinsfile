pipeline {
    agent { label 'ProductionENV' }

    environment {
        INVENTORY   = "hosts"
        PLAYBOOK    = "deploy.yml"
        ANSIBLE_DIR = "Healthify-ProductionPipeline"
    }

        stage('Prepare Workspace') {
            steps {
                deleteDir()
                sh 'git clone https://github.com/SarjakBhandari/Healthify-ProductionPipeline'
            }
        }

        stage('Deploy to Swarm via Ansible') {
            steps {
                dir("${ANSIBLE_DIR}") {
                    sh '''
                        set -e
                        echo ">>> Running Ansible Playbook for Swarm Deployment..."
                        # Optional syntax check before running
                        ansible-playbook --syntax-check -i ${INVENTORY} ${PLAYBOOK}
                        ansible-playbook -i ${INVENTORY} ${PLAYBOOK}
                    '''
                }
            }
        }

    }

    post {
        success {
            mail to: 'sarjakytdfiles@gmail.com',
                 subject: 'HEALTHIFY SWARM DEPLOYMENT SUCCESS',
                 body: """Deployment succeeded ✅

App: http://192.168.50.4:5173
API: http://192.168.50.4:5050

Check services with:
docker service ls

Build URL: ${BUILD_URL}"""
        }
        failure {
            mail to: 'sarjakytdfiles@gmail.com',
                 subject: 'HEALTHIFY SWARM DEPLOYMENT FAILURE',
                 body: """Deployment failed ❌
Check Jenkins logs: ${BUILD_URL}"""
        }
    }
}

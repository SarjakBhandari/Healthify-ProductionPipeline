pipeline {
    agent { label 'ProductionENV' }

    environment {
        INVENTORY   = "hosts"
        PLAYBOOK    = "deploy.yml"
        ANSIBLE_DIR = "Healthify-ProductionPipeline"
    }

    stages {

        stage('Cleanup Old Healthify Containers & Images') {
    steps {
        sh '''
            set -e

            echo ">>> Stopping old Healthify containers..."
            docker ps --filter "name=healthify" --format "{{.ID}}" | xargs -r docker stop

            echo ">>> Removing old Healthify containers..."
            docker ps -a --filter "name=healthify" --format "{{.ID}}" | xargs -r docker rm -f

            echo ">>> Removing old Healthify images..."
            # Only remove healthify-backend or healthify-frontend images, ignore tags from registry
            docker images --format "{{.Repository}}:{{.Tag}}" | grep -E 'healthify-(frontend|backend):' | xargs -r docker rmi -f || true
        '''
    }
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

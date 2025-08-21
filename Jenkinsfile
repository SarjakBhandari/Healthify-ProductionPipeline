pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Image tag to deploy')
        string(name: 'DB_USER', defaultValue: 'postgres', description: 'Database username')
        password(name: 'DB_PASSWORD', defaultValue: 'postgres', description: 'Database password')
        string(name: 'DB_NAME', defaultValue: 'healthify', description: 'Database name')
        string(name: 'POSTGRES_VERSION', defaultValue: '13', description: 'Postgres image version')
        string(name: 'POSTGRES_PORT', defaultValue: '5432', description: 'Postgres exposed port')
        string(name: 'API_PORT', defaultValue: '5050', description: 'Backend API port')
        string(name: 'FRONTEND_PORT', defaultValue: '5173', description: 'Frontend port')
        string(name: 'REGISTRY_IP', defaultValue: '192.168.50.4', description: 'Registry/Swarm Manager IP')
        string(name: 'REGISTRY_PORT', defaultValue: '5000', description: 'Registry port')
    }

    environment {
        INVENTORY = 'inventory.yml'
        PLAYBOOK  = 'deploy_app.yml'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'git@github.com:SarjakBhandari/Healthify-ProductionPipeline.git'
            }
        }

        stage('Deploy to Swarm') {
            steps {
                sh """
                    echo "🚀 Deploying Healthify stack with tag ${IMAGE_TAG}..."
                    ansible-playbook -i ${INVENTORY} ${PLAYBOOK} \
                      --extra-vars "image_tag=${IMAGE_TAG} \
                                    DB_USER=${DB_USER} \
                                    DB_PASSWORD=${DB_PASSWORD} \
                                    DB_NAME=${DB_NAME} \
                                    POSTGRES_VERSION=${POSTGRES_VERSION} \
                                    POSTGRES_PORT=${POSTGRES_PORT} \
                                    API_PORT=${API_PORT} \
                                    FRONTEND_PORT=${FRONTEND_PORT} \
                                    registry_ip=${REGISTRY_IP} \
                                    registry_port=${REGISTRY_PORT}"
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful — Stack running on Swarm"
        }
        failure {
            echo "❌ Deployment failed — check above logs"
        }
    }
}

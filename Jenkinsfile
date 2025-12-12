pipeline {
    agent any
    triggers { githubPush() }
    
    parameters {
        string(name: 'FRONTEND_IMAGE_NAME', defaultValue: 'sohith18/portfolio-frontend', description: 'Docker Image Name')
        string(name: 'FRONTEND_IMAGE_TAG', defaultValue: 'latest', description: 'Docker Image Tag (Build Number)')
        string(name: 'BACKEND_IMAGE_NAME', defaultValue: 'sohith18/portfolio-builder-backend', description: 'Docker Image Name from Backend Job')
        string(name: 'BACKEND_IMAGE_TAG', defaultValue: 'latest', description: 'Docker Image Tag from Backend Job')
        string(name: 'MODEL_IMAGE_NAME', defaultValue: 'sohith18/portfolio-builder-model', description: 'Docker Image Name from Model Job')
        string(name: 'MODEL_IMAGE_TAG', defaultValue: 'latest', description: 'Docker Image Tag from Model Job')
    }

    environment {
        KUBECONFIG = '/opt/jenkins-kube/config'
    }
 
    stages {
        stage ("Deploy to K8s") {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'mongo-uri', variable: 'MONGO_URI'),
                        string(credentialsId: 'secret-key', variable: 'SECRET_KEY'),
                        string(credentialsId: 'hf-token', variable: 'ML_API_KEY')
                    ]) {
                        // Prepare secret JSON strings
                        def backendSecrets = "{\"MONGO_URI\": \"${MONGO_URI}\", \"SECRET_KEY\": \"${SECRET_KEY}\"}"
                        def modelSecrets   = "{\"HF_TOKEN\": \"${ML_API_KEY}\", \"HF_REPO_ID\": \"sohith18/nse-rl-portfolio-builder\"}"


                        sh """
                            echo "Deploying ELK Stack..."
                            ansible-playbook -i inventory.ini elk_playbook.yaml \
                              -e "ansible_python_interpreter=/usr/bin/python3"
                            
                            echo "Deploying Frontend..."
                            ansible-playbook -i inventory.ini frontend_playbook.yaml \
                              -e "ansible_python_interpreter=/usr/bin/python3" \
                              -e "image_name=${params.FRONTEND_IMAGE_NAME}:${params.FRONTEND_IMAGE_TAG}"

                            echo "Deploying Backend..."
                            # PASSING THE IMAGE TAG HERE
                            ansible-playbook -i inventory.ini backend_playbook.yaml \
                              -e "ansible_python_interpreter=/usr/bin/python3" \
                              -e '{"app_secrets": ${backendSecrets}}' \
                              -e "image_name=${params.BACKEND_IMAGE_NAME}:${params.BACKEND_IMAGE_TAG}"

                            echo "Deploying ML Model..."
                            # Assuming you might want to version the model too, otherwise keep as is
                            ansible-playbook -i inventory.ini model_playbook.yaml \
                              -e "ansible_python_interpreter=/usr/bin/python3" \
                              -e '{"app_secrets": ${modelSecrets}}' \
                              -e "image_name=${params.MODEL_IMAGE_NAME}:${params.MODEL_IMAGE_TAG}"

                        """
                    }
                }
            }
        }
    }
}
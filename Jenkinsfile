pipeline {
    agent any
    triggers{githubPush()}

    stages {
        stage ("Deploy to K8s") {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'mongo-uri', variable: 'MONGO_URI'),
                        string(credentialsId: 'secret-key', variable: 'SECRET_KEY'),
                        string(credentialsId: 'hf-token', variable: 'ML_API_KEY')
                    ]) {
                        def backendSecrets = "{\"MONGO_URI\": \"${MONGO_URI}\", \"SECRET_KEY\": \"${SECRET_KEY}\"}"
                        def modelSecrets   = "{\"HF_TOKEN\": \"${ML_API_KEY}\", \"HF_REPO_ID\": \"SiddharthMaramreddy/Portfolio-Builder\"}"

                        sh """
                                echo "Deploying ELK Stack..."
                                ansible-playbook -i inventory.ini elk_playbook.yaml \
                                  -e "ansible_python_interpreter=/usr/bin/python3"
                                
                                echo "Deploying Frontend..."
                                ansible-playbook -i inventory.ini frontend_playbook.yaml \
                                  -e "ansible_python_interpreter=/usr/bin/python3"

                                echo "Deploying Backend..."
                                ansible-playbook -i inventory.ini backend_playbook.yaml \
                                  -e "ansible_python_interpreter=/usr/bin/python3" \
                                  -e '{"app_secrets": ${backendSecrets}}'

                                echo "Deploying ML Model..."
                                ansible-playbook -i inventory.ini model_playbook.yaml \
                                  -e "ansible_python_interpreter=/usr/bin/python3" \
                                  -e '{"app_secrets": ${modelSecrets}}'
                            """
                    }
                }
            }
        }
    }
}
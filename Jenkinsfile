pipeline {
    agent any

    parameters {
        string(name: 'TARGET_HOSTS', defaultValue: '192.168.56.24,192.168.56.25,192.168.56.26', description: 'Comma separated IP Addresses')
        string(name: 'DB_ROOT_PASS', defaultValue: 'MySecurePass123!', description: 'Root Password')
        string(name: 'APP_USER', defaultValue: 'app_user', description: 'App User')
        string(name: 'APP_PASS', defaultValue: 'AppPass789!', description: 'App Password')
        
        text(name: 'SQL_CONFIG_JSON', description: 'MariaDB Config', defaultValue: '''{
  "user": "mysql",
  "port": "3306",
  "innodb_buffer_pool_size": "512M",
  "max_connections": "100"
}''')

        text(name: 'GALERA_CONFIG_JSON', description: 'Galera Config', defaultValue: '''{
  "wsrep_on": "ON",
  "wsrep_sst_method": "rsync",
  "wsrep_cluster_name": "my_prod_cluster"
}''')
    }

    environment {
        SSH_KEY_PATH = '/var/lib/jenkins/.ssh/ansible_key' 
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }

    stages {
        stage('Prepare Workspace') {
            steps {
                sh 'sudo chown -R jenkins:jenkins .'
            }
        }

        stage('Generate Configs') {
            steps {
                script {
                    def ips = params.TARGET_HOSTS.split(',')
                    
                    sh "echo '[mariadb_nodes]' > inventory.ini"
                    for (ip in ips) { 
                        sh "echo '${ip.trim()}' >> inventory.ini" 
                    }
                    
                    sh """
                        echo '' >> inventory.ini
                        echo '[all:vars]' >> inventory.ini
                        echo 'ansible_user=user' >> inventory.ini
                        echo 'ansible_ssh_private_key_file=${SSH_KEY_PATH}' >> inventory.ini
                        echo 'ansible_python_interpreter=/usr/bin/python3.10' >> inventory.ini
			echo 'ansible_become_timeout=60' >> inventory.ini
                    """

                    def payload = """
                    {
                      "sql_final_config": ${params.SQL_CONFIG_JSON},
                      "galera_final_config": ${params.GALERA_CONFIG_JSON},
                      "wsrep_provider_options_string": "gcache.size=512M;gcache.recover=ON",
                      "mysql_root_password": "${params.DB_ROOT_PASS}",
                      "app_user_name": "${params.APP_USER}",
                      "app_user_password": "${params.APP_PASS}"
                    }
                    """
                    
                    writeFile file: 'pipeline_inputs.json', text: payload
                }
            }
        }

        stage('Run Ansible') {
            steps {
                sh """
                    sudo -E ansible-playbook -i inventory.ini playbook3.yml \
                    --extra-vars "@pipeline_inputs.json"
                """
            }
        }
    }
}

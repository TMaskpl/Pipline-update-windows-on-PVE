def FAILED_STAGE

pipeline {
    agent any
    parameters {
        string defaultValue: 'pve-host', description: 'Podaj nazwe hosta PVE', name: 'host_vm'
        string defaultValue: '666', description: 'Podaj id vm', name: 'vm_id'
        string defaultValue: 'win-host', description: 'Podaj nazwe hosta Windows', name: 'win_vm'
        booleanParam defaultValue: true, description: 'Aktualizacja Windows', name: 'run_update'
        booleanParam defaultValue: true, description: 'Tworzenie snapshot', name: 'create_snap'
        booleanParam defaultValue: false, description: 'Czy skasować snap po aktualizacji', name: 'remove_snap'
    }
    options {
        ansiColor('vga')
    }
    stages {
        stage('Git clone...') {
            steps {
                script {
                    FAILED_STAGE=env.STAGE_NAME
                    echo 'Clone repository...'
                    git branch: 'main', url: 'https://github.com/TMaskpl/Pipline-update-windows-on-PVE.git'
                }
            }
        }

        stage('Check system...') {
            steps {
                script {
                    FAILED_STAGE=env.STAGE_NAME
                    echo 'Check system...'
                    ansiblePlaybook installation: 'Ansible', extras: '-e host_vm=${host_vm}', playbook: 'check-system.yml', tags: 'check'            
                }
            }        
        }

        stage('Create snapshot...') {
            when {
                expression { 
                    return params.create_snap
                }
            }
            steps {
                script {
                    FAILED_STAGE=env.STAGE_NAME
                    echo 'Create snapshot...'
                    ansiblePlaybook installation: 'Ansible', extras: '-e host_vm=${host_vm} -e vm_id=${vm_id}', playbook: 'create_snapshot.yml', tags: 'snap'
                }
            }
        }
        
        stage('Update Windows ...') {
            when {
                expression { 
                    return params.run_update
                }
            }
            steps {
                script {
                    FAILED_STAGE=env.STAGE_NAME
                    echo 'Update Windows...'
                    ansiblePlaybook installation: 'Ansible', extras: '-e win_vm=${win_vm}', playbook: 'windows_update.yml', tags: 'updates'
                }
            }
        }

        stage('Remove snapshot...') {
            when {
                expression { 
                    return params.remove_snap
                }
            }
            steps {
                script {
                    FAILED_STAGE=env.STAGE_NAME
                    echo 'Remove snapshot...'
                    ansiblePlaybook installation: 'Ansible', extras: '-e host_vm=${host_vm} -e vm_id=${vm_id}', playbook: 'remove_snapshot.yml', tags: 'remove'
                }
            }
        }
    }

    post {
        failure {
            echo "Failed stage name: ${FAILED_STAGE}"
            slackSend (color: '#FF0000', channel: 'tmaskpl', message: """FAILED_STAGE: Job "${env.STAGE_NAME} ${env.JOB_NAME} [${env.BUILD_NUMBER}]" (${env.BUILD_URL})""")
        }
    }
}

@Library('etn-ipm2-jenkins@push-to-mbt') _

pipeline {
    agent {
        docker {
            label 'docker-dev-1'
            image infra.getDockerAgentImage()
            args '--oom-score-adj=100 -v /etc/ssh/id_rsa_git-proxy-cache:/etc/ssh/id_rsa_git-proxy-cache:ro -v /etc/ssh/ssh_config:/etc/ssh/ssh_config:ro -v /etc/gitconfig:/etc/gitconfig:ro'
        }
    }
    
    stages {
        stage ('test') {
            steps {
                echo "ok"
            }
        }
    }
}

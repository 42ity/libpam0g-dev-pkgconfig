@Library('etn-ipm2-jenkins') _

pipeline {
    agent {
        docker {
            label 'docker-dev-1'
            image infra.getDockerAgentImage()
            args '--oom-score-adj=100 -v /etc/ssh/id_rsa_git-proxy-cache:/etc/ssh/id_rsa_git-proxy-cache:ro -v /etc/ssh/ssh_config:/etc/ssh/ssh_config:ro -v /etc/gitconfig:/etc/gitconfig:ro'
        }
    }
    parameters {
        // Use DEFAULT_DEPLOY_BRANCH_PATTERN and DEFAULT_DEPLOY_JOB_NAME if
        // defined in this jenkins setup -- in Jenkins Management Web-GUI
        // see Configure System / Global properties / Environment variables
        // Default (if unset) is empty => no deployment attempt after good test
        // See zproject Jenkinsfile-deploy.example for an example deploy job.
        // TODO: Try to marry MultiBranchPipeline support with pre-set defaults
        // directly in MultiBranchPipeline plugin, or mechanism like Credentials,
        // or a config file uploaded to master for all jobs or this job, see
        // https://jenkins.io/doc/pipeline/examples/#configfile-provider-plugin
        string (
            defaultValue: '${DEFAULT_DEPLOY_BRANCH_PATTERN}',
            description: 'Regular expression of branch names for which a deploy action would be attempted after a successful build and test; leave empty to not deploy. Reasonable value is ^(master|release/.*|feature/*)$',
            name : 'DEPLOY_BRANCH_PATTERN')
        string (
            defaultValue: '${DEFAULT_DEPLOY_JOB_NAME}',
            description: 'Name of your job that handles deployments and should accept arguments: DEPLOY_GIT_URL DEPLOY_GIT_BRANCH DEPLOY_GIT_COMMIT -- and it is up to that job what to do with this knowledge (e.g. git archive + push to packaging); leave empty to not deploy',
            name : 'DEPLOY_JOB_NAME')
        booleanParam (
            defaultValue: true,
            description: 'If the deployment is done, should THIS job wait for it to complete and include its success or failure as the build result (true), or should it schedule the job and exit quickly to free up the executor (false)',
            name: 'DEPLOY_REPORT_RESULT')
        booleanParam (
            defaultValue: true,
            description: 'Require that there are no files not discovered changed/untracked via .gitignore after builds and tests?',
            name: 'REQUIRE_GOOD_GITIGNORE')
        booleanParam (
            defaultValue: true,
            description: 'When using temporary subdirs in build/test workspaces, wipe them after successful builds?',
            name: 'DO_CLEANUP_AFTER_BUILD')
    }
    triggers {
        pollSCM 'H/5 * * * *'
    }
    options {
        disableConcurrentBuilds()
        // Jenkins community suggested that instead of a default checkout, we can do
        // an explicit step for that. It is expected that either way Jenkins "should"
        // record that a particular commit is being processed, but the explicit ways
        // might work better. In either case it honors SCM settings like refrepo if
        // set up in the Pipeline or MultiBranchPipeline job.
        skipDefaultCheckout()
    }
    stages {
        stage ('pre-clean') {
                    steps {
                        milestone ordinal: 20, label: "${env.JOB_NAME}@${env.BRANCH_NAME}"
                        dir("tmp") {
                            sh 'if [ -s Makefile ]; then make -k distclean || true ; fi'
                            sh 'chmod -R u+w .'
                            deleteDir()
                        }
                        sh 'rm -f ccache.log cppcheck.xml'
                    }
        }
        stage ('git') {
                    steps {
                        retry(3) {
                            checkout scm
                        }
                        milestone ordinal: 30, label: "${env.JOB_NAME}@${env.BRANCH_NAME}"
                        script {
                            buildenv.setExtraEnvVariables()
                            buildenv.listInstalledPackage()
                            buildenv.checkDebianBuildDependencies()
                        }
                    }
        }
        stage ('Deploy') {
            parallel {
                stage ("Push to OBS") {
                    when {
                        anyOf {
                            branch 'master'
                            branch "release/*"
                            branch "featureimage/*"
                            branch 'FTY'
                            branch '*-FTY-master'
                            branch '*-FTY'
                        }
                    }
                    steps {
                        script {
                            deploy.pushToOBS()
                        }
                    }
                }
            }
        }
        stage ('cleanup') {
            when { expression { return ( params.DO_CLEANUP_AFTER_BUILD ) } }
            steps {
                deleteDir()
            }
        }
    }
    post {
        success {
            script {
                if (currentBuild.getPreviousBuild()?.result != 'SUCCESS') {
                    // Uncomment desired notification

                    //slackSend (color: "#008800", message: "Build ${env.JOB_NAME} is back to normal.")
                    //emailext (to: "qa@example.com", subject: "Build ${env.JOB_NAME} is back to normal.", body: "Build ${env.JOB_NAME} is back to normal.")
                }
            }
        }
        failure {
            // Uncomment desired notification
            // Section must not be empty, you can delete the sleep once you set notification
            sleep 1
            //slackSend (color: "#AA0000", message: "Build ${env.BUILD_NUMBER} of ${env.JOB_NAME} ${currentBuild.result} (<${env.BUILD_URL}|Open>)")
            //emailext (to: "qa@example.com", subject: "Build ${env.JOB_NAME} failed!", body: "Build ${env.BUILD_NUMBER} of ${env.JOB_NAME} ${currentBuild.result}\nSee ${env.BUILD_URL}")
        }
    }
}

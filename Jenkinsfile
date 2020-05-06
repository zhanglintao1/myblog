@Library('luffy-devops') _

pipeline {
    agent { label 'jnlp-slave'}
    
    options {
		buildDiscarder(logRotator(numToKeepStr: '10'))
		disableConcurrentBuilds()
		timeout(time: 20, unit: 'MINUTES')
		gitLabConnection('gitlab')
	}

    environment {
        IMAGE_REPO = "172.21.32.15:5000/demo/myblog"
        DINGTALK_CREDS = credentials('dingTalk')
        IMAGE_CREDENTIAL = "credential-registry"
    }

    stages {
        stage('git-log') {
            steps {
                script{
                    sh "git log --oneline -n 1 > gitlog.file"
                    env.GIT_LOG = readFile("gitlog.file").trim()
                }
                sh 'printenv'
            }
        }        
        stage('checkout') {
            steps {
                container('tools') {
                    checkout scm
                }
                updateGitlabCommitStatus(name: env.STAGE_NAME, state: 'success')
                script{
                    env.BUILD_TASKS = env.STAGE_NAME + "√..." + env.TAB_STR
                }
            }
        }
        stage('hello-devops') {
            steps {
                script {
                    devops.hello("树哥").sayHi().answer().sayBye()
                }
            }
        }
        stage('build-image') {
            steps {
                container('tools') {
                    script{
                        devops.dockerBuild(
                            "Dockerfile",
                            ".",
                            "${IMAGE_REPO}",
                            "${GIT_COMMIT}",
                            IMAGE_CREDENTIAL,
                        ).start().push()
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                container('tools') {
                    script{
                    	devops.deployMulti("deploy").start()
                    }
                }
            }
        }
    }
    post {
        success {
            script {
                container('tools') {
                    devops.notification("myblog", "luffy,流水线完成了", "${GIT_COMMIT}","dingTalk").success()
                }
            }
        }
        failure {
            script {
                container('tools') {
                    devops.notification("myblog", "luffy,流水线完成了", "${GIT_COMMIT}", "dingTalk").failure()
                }
            }
        }
    }
}
pipeline {
    agent none


        stages {
            stage('worker-buld') {
            agent {
            docker{
                image "maven:3.6.1-jdk-8-alpine"
                args '-v $HOME/.m2:/root/.m2'
                    }
                }
            when {
            changeset "**/worker/**"}
            
                steps {
                    echo 'compiling worker app'
                    dir('worker'){
                sh "mvn compile"
                }
                }
            }
            stage('worker-test') {
            agent {
            docker{
                image "maven:3.6.1-jdk-8-alpine"
                args '-v $HOME/.m2:/root/.m2'
            }
            }
            when {
            changeset "**/worker/**"}
            
                steps {
                    echo 'running tests on worker app'
            dir ('worker'){
                sh "mvn clean test"	}                
                    }
                }
            stage('worker-docker-package'){
            agent any
            when {
            
            changeset "**/worker/**"
            branch 'master'
            }
                steps {
                    echo 'packaging worker app with docker'
                    script {
                        docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
                            def workerImage = docker.build("jlargaespada/worker:v${env.BUILD_ID}", "./worker")
                            workerImage.push()
                            workerImage.push("${env.BRANCH_NAME}")
                            workerImage.push("latest")
                        }
                    }
                    }
                }
            stage('vote-build') {
                    agent {
            docker{
                image "python:2.7.16-slim"
                args "--user root"
            }
            }
                when {
                changeset "**/vote/**"
                branch 'master'
                }
                    
                steps {
                    echo 'Installing python pip for vote app'
                    dir('vote'){
                            sh "pip install -r requirements.txt"
                    }
                }
            }
            stage('vote-test') {
                    agent {
            docker{
                image "python:2.7.16-slim"
                args "--user root"
            }
            }
                when {
                changeset "**/vote/**"
                }
                    
                steps {
                    echo 'Installing and testing python pip for vote app'
                    dir ('vote'){
                        sh "pip install -r requirements.txt"
                        sh "nosetests -v"
                    }
                }
            }
            stage('vote-docker-package') {
            agent any
                when {
                changeset "**/vote/**"
                }
                    
                steps {
                    echo 'Installing and testing python pip for vote app'
                    script {
                        docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
                            def voteImage = docker.build("jlargaespada/vote:v${env.BUILD_ID}", "./vote")
                            voteImage.push()
                            voteImage.push("${env.BRANCH_NAME}")
                            voteImage.push("latest")
                        }
                    }
                }
            }
            stage('result-build') {
                    agent {
            docker{
                image "node:8.16.0-alpine"
            }
            }
                when {
                changeset "**/result/**"
                }
                    
                steps {
                    echo 'Installing npm for result app'
                    dir('result'){
                            sh "npm install"
                    }
                }
            }
            stage('result-test') {
            agent {
            docker{
                image "node:8.16.0-alpine"
            }
            }
                when {
                changeset "**/result/**"
                }
                    
                steps {
                    echo 'Installing and testing npm for result app'
                    dir ('result'){
                        sh "npm install && npm test"
                    }
                }
            }
                    stage('result-docker-package') {
                
                when {
                changeset "**/result/**"
                branch 'master'
                }
                    
                steps {
                    echo 'Installing and testing npm for result app'
                    script {
                        docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
                            def resultImage = docker.build("jlargaespada/result:v${env.BUILD_ID}", "./result")
                            resultImage.push()
                            resultImage.push("${env.BRANCH_NAME}")
                            resultImage.push("latest")
                        }
                    }
                }
            }

            stage('deploy to dev'){
                agent any
                when{
                    branch 'master'
                }
                steps{
                    echo 'Deploy instavote app with docker compose'
                    sh 'docker-compose up -d'
                }
            }

        }
 
    post {
        always {
            echo "Pipeline for InstaApp run is complete.."
        }
        failure {
		slackSend (channel: "test-jenkins", message: "Build failure - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success {
		slackSend (channel: "test-jenkins", message: "Build succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
    }
}

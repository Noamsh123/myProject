pipeline {
    agent any

    stages{
        stage ("checkout") {
            steps{
                deleteDir()
                checkout scm
            }
        }

        stage ('build') {
            steps{
                sh "docker build -t shamir-repo ."
            }
        }

        stage ('run and test'){
            steps{
                sh "docker-compose up -d"
                sleep 16
                sh "bash tests/e2e.sh"
            }
            post{
                always{
                    sh "docker-compose down"
                }
            }
        }
        stage ("publish release"){
            when{
                expression{
                    return GIT_BRANCH.contains('release/')
                }
            }
            steps{
                script{
                    version=sh (script: "bash release/version.sh",
                    returnStdout: true).trim()

                    docker.withRegistry("https://644435390668.dkr.ecr.eu-west-2.amazonaws.com/shamir-repo","ecr:eu-west-2:my-aws-access"){
                        sh "docker tag shamir-repo:latest 644435390668.dkr.ecr.eu-west-2.amazonaws.com/shamir-repo:${version}.${BUILD_NUMBER}"
                        sh "docker push 644435390668.dkr.ecr.eu-west-2.amazonaws.com/shamir-repo:${version}.${BUILD_NUMBER}"
                    }
                }
            }
            
        }


        stage ("publish"){
            when{
                expression{
                    return GIT_BRANCH.contains('main')
                }
            }
            steps {
                script{
                    docker.withRegistry("https://644435390668.dkr.ecr.eu-west-2.amazonaws.com/shamir-repo","ecr:eu-west-2:my-aws-access"){
                        sh "docker tag shamir-repo:latest 644435390668.dkr.ecr.eu-west-2.amazonaws.com/shamir-repo:1.1.${BUILD_NUMBER}"
                        sh "docker push 644435390668.dkr.ecr.eu-west-2.amazonaws.com/shamir-repo:1.1.${BUILD_NUMBER}"
                    }
                }
            }
        }
        stage("deploy"){
            when{
                expression{
                    return GIT_BRANCH.contains('main')
                }
            }
            steps {
                sleep 10
                sh "./deploy.sh ${BUILD_NUMBER}"
            }
        }
    }
}
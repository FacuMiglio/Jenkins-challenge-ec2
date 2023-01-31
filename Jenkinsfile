pipeline {
    agent any
    environment {
        EC2INSTANCEDEV = 'ec2-user@44.201.56.64'
        EC2INSTANCETST = 'ec2-user@44.203.20.13'
        EC2INSTANCEPRD = 'ec2-user@34.201.35.162'
        APPNAME = 'hello-bootcamp-app'
        REGISTRY = 'facumiglio'
        DOCKER_HUB_LOGIN = credentials('docker-grupo1')
        IMAGENAME = 'hello-bootcamp'
    }
    stages {
        stage('Init') {
            agent{
                docker {
                    image 'node:erbium-alpine'
                    args '-u root:root'
                }
            }
            steps {
                echo "Init"
                sh 'npm install'
            }
        }
        stage('Test') {
            agent{
                docker {
                    image 'node:erbium-alpine'
                    args '-u root:root'
                }
            }
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                sh 'npm run test'
                }
            }
        }
        stage('Build') {
            steps {
                echo 'Stage Build'
                sh 'docker build -t ${APPNAME}:${BUILD_NUMBER} .'
                sh 'docker tag ${APPNAME}:${BUILD_NUMBER} ${REGISTRY}/${APPNAME}:${BUILD_NUMBER}'
            }
        }
        stage('Push to Registry') {
            steps {
                echo 'Stage Push'
                sh 'docker login --username=$DOCKER_HUB_LOGIN_USR --password=$DOCKER_HUB_LOGIN_PSW'
                sh 'docker push ${REGISTRY}/${APPNAME}:${BUILD_NUMBER}'
                
            }
        }
        stage('Deploy PRD') {
            steps {
                echo 'Stage Deploy dev'
                sh ("sed -i -- 's/REGISTRY/$REGISTRY/g' docker-compose.yml")
                sh ("sed -i -- 's/APPNAME/$APPNAME/g' docker-compose.yml")
                sh ("sed -i -- 's/IMAGENAME/$IMAGENAME/g' docker-compose.yml")
                sh ("sed -i -- 's/TAG/$BUILD_NUMBER/g' docker-compose.yml")
                sshagent(['ssh-ec2']){
                 sh 'scp -o StrictHostKeyChecking=no docker-compose.yml ${EC2INSTANCEPRD}:/home/ec2-user' 
                 sh 'ssh ${EC2INSTANCEPRD} ls -lrt'
                 sh 'ssh ${EC2INSTANCEPRD} docker-compose up -d'
                }
            }
        }
        stage('Notification') {
            steps {
                echo 'Stage Notify'
                echo 'El mejor Team, el GRUPO1'
            }
        }
    }
}
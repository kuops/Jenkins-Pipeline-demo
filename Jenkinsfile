pipeline {
    agent any
    options {
        timestamps()
    }
    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'master', description: 'default build branch')
        booleanParam(name: 'RUN_SONAR_SCANNER', defaultValue: true, description: 'run the sonar scanner check.')
    }
    environment {
        MAVEN_IMAGE = 'maven:3-alpine'
        SONAR_SCANNER_IMAGE = 'cnlinux/sonar-scanner:3.0.3'
        SONAR_SERVER = 'http://10.0.7.1:9000'
        DOCKER_REGISTRY = "10.0.7.1:5000"
        APP_NAME = 'jenkins-jipeline-demo'
        DEPLOY_HOST = '10.0.7.1:2376'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.GIT_BRANCH}", url: 'https://github.com/opspy/Jenkins-Pipeline-demo.git'
            }
        }
        stage('Test') {
            parallel {
                stage ('Unit Test') {
                    agent {
                        docker {
                            reuseNode true
                            image '${MAVEN_IMAGE}'
                            args '-v $HOME/.m2:/root/.m2'
                        }
                    }
                    steps {
                        sh 'mvn test'
                        junit '**/target/**/*.xml'
                    }
                }
                stage ('Sonar Scanner') {
                    when {
                        environment name: 'RUN_SONAR_SCANNER', value: 'true'
                    }
                    agent {
                        docker {
                            reuseNode true
                            image '${SONAR_SCANNER_IMAGE}'
                        }
                    }
                    steps {
                        sh 'sonar-scanner -Dsonar.host.url=${SONAR_SERVER}'
                    }
                }
            }
        }
       stage('Build War') {
            agent {
                docker {
                    reuseNode true
                    image '${MAVEN_IMAGE}'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn -Dmaven.test.skip=true clean install'
            }
       }
       stage('Docker image') {
            steps {
                sh """
                        mv -f target/*.war deployment/
                        docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER} deployment
                        docker push ${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}
                        docker rmi ${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}
                        rm -f deployment/*.war
                """
            }
        }
        stage('Deploy') {
            steps {
                input message: 'Are you sure Deployment?', ok: 'Yes'
                sh"""
                    docker -H ${DEPLOY_HOST} rm -f ${APP_NAME} | true
                    docker -H ${DEPLOY_HOST} run -d --name ${APP_NAME} -p 9090:8080 ${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}
                """
                }
        }
    }
    post {
        always {
            emailext body: """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
            subject: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'", 
            to: 'admin@example.com'
        }
    }
}

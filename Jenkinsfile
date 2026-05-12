pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub_credentials')
        DOCKER_IMAGE = '2624416562/teedy'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', env.DOCKER_HUB_CREDENTIALS) {
                        docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").push()
                        docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").push('latest')
                    }
                }
            }
        }

        stage('Run Three Containers') {
            steps {
                script {
                    sh 'docker rm -f teedy_8082 teedy_8083 teedy_8084 || true'
                    docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").run('-d -p 8082:8080 --name teedy_8082')
                    docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").run('-d -p 8083:8080 --name teedy_8083')
                    docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").run('-d -p 8084:8080 --name teedy_8084')
                }
            }
        }
    }

    post {
        success {
            echo '✅ 成功！三个容器运行在 8082,8083,8084'
        }
        failure {
            echo '❌ 失败，请检查日志'
        }
    }
}
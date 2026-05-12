pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub_credentials')
        DOCKER_IMAGE = '2624416562/teedy'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        // Maven 构建 - 保留你原有的
        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        // 构建 Docker 镜像
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}")
                }
            }
        }

        // 推送到 Docker Hub
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

        // 运行三个容器
        stage('Run Three Containers') {
            steps {
                script {
                    // 清理旧容器
                    sh 'docker rm -f teedy_8082 teedy_8083 teedy_8084 || true'
                    
                    // 运行三个容器
                    docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").run('-d -p 8082:8080 --name teedy_8082')
                    docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").run('-d -p 8083:8080 --name teedy_8083')
                    docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").run('-d -p 8084:8080 --name teedy_8084')
                }
            }
        }
    }

    post {
        success {
            echo '✅ 流水线成功！三个容器运行在 8082,8083,8084'
        }
        failure {
            echo '❌ 流水线失败，请检查日志'
        }
    }
}
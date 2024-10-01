pipeline {
    agent { label 'vm2' }
    environment {
        APP_NAME = "web-api"
        IMAGE_NAME = 'spdx'

        ROBOT_REPO = 'https://github.com/kitti-best/dont-look-its-test-repo-for-testing-another-repo'
        ROBOT_BRANCH = 'main'
        ROBOT_FILE = 'robot_test.robot'

        MAIN_REPO = 'https://github.com/kitti-best/jenkins-assignment'
        MAIN_BRANCH = 'feature'

        NAMESPACE = 'kitti-best'
        GITHUB_CRED = credentials('user_01')
    }

    stages {
        stage("Build") {
            steps {
                sh "pwd"
                sh "docker build --tag ${IMAGE_NAME} ."
                sh "docker image ls"
            }
        }

        stage("Run and Test"){
            steps {
                ////// update and start container //////
                script {
                    sh(script: "docker stop ${APP_NAME}", returnStatus: true)
                    sh(script: "docker rm ${APP_NAME} -f", returnStatus: true)

                    sh(script: "docker run --name ${APP_NAME} -d -p 5000:5000 ${IMAGE_NAME}")
                }

                ////// unit test running batch  //////
                script {
                    // if not OK
                    if (sh(script: "docker exec ${APP_NAME} sh -c 'python -m unit_test -v; exit;'", returnStatus: true)) {
                        error("Build terminated: failed the unit test'.")
                    }
                }
                /////////////////////////////////////////

                ////// unit test running batch  //////
                git url: "${ROBOT_REPO} ./robot", branch: "${ROBOT_BRANCH}"
                sh "robot ${ROBOT_FILE}"
                //////////////////////////////////////
            }
        }

        stage("Release") {
            steps {
                sh "docker tag ${IMAGE_NAME} ghcr.io/${NAMESPACE}/${IMAGE_NAME}"
                sh "docker tag ${IMAGE_NAME} ghcr.io/${NAMESPACE}/${IMAGE_NAME}:${BUILD_NUMBER}"

                sh "docker login ghcr.io -u ${GITHUB_CRED_USR} -p ${GITHUB_CRED_PSW}"

                sh "docker push ghcr.io/${NAMESPACE}/${IMAGE_NAME}"
                sh "docker push ghcr.io/${NAMESPACE}/${IMAGE_NAME}:${BUILD_NUMBER}"

                sh "docker inspect ghcr.io/${NAMESPACE}/${IMAGE_NAME}"
                sh "docker inspect ghcr.io/${NAMESPACE}/${IMAGE_NAME}:${BUILD_NUMBER}"

                sh "docker rmi ghcr.io/${NAMESPACE}/${IMAGE_NAME}"
                sh "docker rmi ghcr.io/${NAMESPACE}/${IMAGE_NAME}:${BUILD_NUMBER}"
            }
        }

        stage("Deploy") {
            agent { label 'vm3' }
            steps {
                sh "pwd"
                sh "docker login ghcr.io -u ${GITHUB_CRED_USR} -p ${GITHUB_CRED_PSW}"
                sh "docker pull ghcr.io/${NAMESPACE}/${IMAGE_NAME}"

                sh returnStatus: true, script: "docker stop ${APP_NAME}"
                sh returnStatus: true, script: "docker rm ${APP_NAME} -f"
                sh "docker run --name ${APP_NAME} -d -p 5000:5000 ghcr.io/${NAMESPACE}/${IMAGE_NAME}"
            }
        }

    }
}
pipeline {
    agent any  // 使用任何可用的 agent（这里只是用来运行 shell 命令）
    environment {
        DEV_DB_URL = 'jdbc:mysql://host.docker.internal:3306/dev1_db'
        PROD_DB_URL = 'jdbc:mysql://host.docker.internal:3306/prod_db'
        DB_USER = 'flyway_demo'
        DB_PASSWORD = 'demo123'
        MIGRATION_LOC = '/flyway/project/migrations'  // 容器内的路径
        WORKSPACE = "${WORKSPACE}"  // Jenkins 内置变量，指向当前工作目录
    }
    stages {
        stage('Check Migration Status') {
            steps {
                script {
                    sh """
                        docker run --platform linux/amd64 --rm \
                            -v ${WORKSPACE}:/flyway/project \
                            redgate/flyway \
                            flyway -url=${DEV_DB_URL} \
                                   -user=${DB_USER} \
                                   -password=${DB_PASSWORD} \
                                   -locations=filesystem:${MIGRATION_LOC} \
                                   info
                    """
                }
            }
        }
        stage('Migrate Dev Database') {
            steps {
                script {
                    sh """
                        docker run --platform linux/amd64 --rm \
                            -v ${WORKSPACE}:/flyway/project \
                            redgate/flyway \
                            flyway -url=${DEV_DB_URL} \
                                   -user=${DB_USER} \
                                   -password=${DB_PASSWORD} \
                                   -locations=filesystem:${MIGRATION_LOC} \
                                   migrate
                    """
                }
            }
        }
        stage('Generate Change Report (Optional)') {
            steps {
                script {
                    sh """
                        docker run --platform linux/amd64 --rm \
                            -v ${WORKSPACE}:/flyway/project \
                            redgate/flyway \
                            flyway -url=${PROD_DB_URL} \
                                   -user=${DB_USER} \
                                   -password=${DB_PASSWORD} \
                                   -locations=filesystem:${MIGRATION_LOC} \
                                   prepare -outputType=html > change_report.html
                    """
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'change_report.html', allowEmptyArchive: true
                }
            }
        }
        stage('Deploy to Production (Manual Approval)') {
            input {
                message "批准部署到生产环境？"
                ok "批准"
            }
            steps {
                script {
                    sh """
                        docker run --platform linux/amd64 --rm \
                            -v ${WORKSPACE}:/flyway/project \
                            redgate/flyway \
                            flyway -url=${PROD_DB_URL} \
                                   -user=${DB_USER} \
                                   -password=${DB_PASSWORD} \
                                   -locations=filesystem:${MIGRATION_LOC} \
                                   migrate
                    """
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline 执行完成！'
        }
    }
}

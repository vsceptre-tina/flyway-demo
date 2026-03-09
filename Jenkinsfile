pipeline {
    agent {
        docker {
            image 'redgate/flyway'                     // 官方 Flyway 镜像
            args '-v $PWD:/flyway/project'            // 挂载项目目录到容器
        }
    }
    environment {
        // 数据库连接信息（从 Jenkins 凭据读取，避免明文）
        // 这里先用明文演示，后续可改为凭据
        DEV_DB_URL = 'jdbc:mysql://host.docker.internal:3306/dev1_db'
        PROD_DB_URL = 'jdbc:mysql://host.docker.internal:3306/prod_db'
        DB_USER = 'flyway_demo'
        DB_PASSWORD = 'demo123'
        MIGRATION_LOC = 'filesystem:/flyway/project/migrations'
    }
    stages {
        stage('Check Migration Status') {
            steps {
                sh """
                    flyway -url=${DEV_DB_URL} \\
                           -user=${DB_USER} \\
                           -password=${DB_PASSWORD} \\
                           -locations=${MIGRATION_LOC} \\
                           info
                """
            }
        }
        stage('Migrate Dev Database') {
            steps {
                sh """
                    flyway -url=${DEV_DB_URL} \\
                           -user=${DB_USER} \\
                           -password=${DB_PASSWORD} \\
                           -locations=${MIGRATION_LOC} \\
                           migrate
                """
            }
        }
        stage('Generate Change Report (Optional)') {
            steps {
                sh """
                    flyway -url=${PROD_DB_URL} \\
                           -user=${DB_USER} \\
                           -password=${DB_PASSWORD} \\
                           -locations=${MIGRATION_LOC} \\
                           prepare -outputType=html > change_report.html
                """
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
                sh """
                    flyway -url=${PROD_DB_URL} \\
                           -user=${DB_USER} \\
                           -password=${DB_PASSWORD} \\
                           -locations=${MIGRATION_LOC} \\
                           migrate
                """
            }
        }
    }
    post {
        always {
            echo 'Pipeline 执行完成！'
        }
    }
}

pipeline {

    agent any


    environment {

        SERVER = "192.168.40.117"
        SSH_USER = "nik"

        TELEGRAM_TOKEN = credentials('telegram-token')
        TELEGRAM_CHAT_ID = credentials('telegram-chat-id')

    }


    stages {


        stage('Deploy Monitoring Stack') {

            steps {

                echo "🚀 Deploy monitoring stack"


                sh '''
                ssh -o StrictHostKeyChecking=no ${SSH_USER}@${SERVER} \
                "bash /opt/scripts/deploy-monitoring-test.sh"
                '''

            }

        }


        stage('Health Check') {

            steps {

                echo "🔍 Checking monitoring services"


                sh '''
                ssh -o StrictHostKeyChecking=no ${SSH_USER}@${SERVER} '

                echo "Checking containers..."

                docker ps | grep prometheus-test
                docker ps | grep grafana-test
                docker ps | grep node-exporter-test


                echo "Checking Prometheus..."

                curl -f http://localhost:9190/-/healthy


                echo "Checking Grafana..."

                curl -f http://localhost:3300/api/health


                echo "Checking Node Exporter..."

                curl -f http://localhost:9110/metrics > /dev/null


                echo "All services are OK"

                '
                '''

            }

        }


    }


    post {


        success {

            sh """
            curl -s -X POST \
            "https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage" \
            -d chat_id=${TELEGRAM_CHAT_ID} \
            --data-urlencode "text=✅ Server Health Monitor

🚀 Deploy: SUCCESS

📊 Monitoring Stack:

🟢 Prometheus: UP
🟢 Grafana: UP
🟢 Node Exporter: UP

🖥 Server:
${SERVER}

🔧 Jenkins Build:
${BUILD_NUMBER}

⏱ Duration:
${currentBuild.durationString}"
            """

        }



        failure {

            sh """
            curl -s -X POST \
            "https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage" \
            -d chat_id=${TELEGRAM_CHAT_ID} \
            --data-urlencode "text=❌ Server Health Monitor

🔥 Deploy: FAILED

🖥 Server:
${SERVER}

🔧 Jenkins Build:
${BUILD_NUMBER}

Check Jenkins logs"
            """

        }


    }


}

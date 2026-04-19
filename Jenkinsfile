pipeline {
    // 1. CHÌA KHÓA ĐIỀU HƯỚNG: Gọi đúng tên nhãn của con máy ảo
    agent { label 'jmeter-slave' }

    parameters {
        string(name: 'SCRIPT_FILENAME', defaultValue: 'Flow_01_Login.jmx', description: 'Tên file kịch bản trong thư mục scripts/')
    }

    environment {
        // 2. ÔNG PHẢI SỬA DÒNG NÀY: Trỏ tới thư mục bin của JMeter TRÊN MÁY ẢO LINUX
        // Ví dụ: /home/xuanle/apache-jmeter-5.6.3/bin hoặc /opt/apache-jmeter-5.6.3/bin
        JMETER_HOME = "/Đường/dẫn/đến/jmeter/bin/của/máy/ảo"
    }

    stages {
        stage('Prepare & Checkout') {
            steps {
                // Khi Job bắt đầu, Jenkins đã ngầm tự động pull code từ Git về rồi.
                // Đoạn này chỉ in ra log để ông dễ theo dõi thôi.
                echo "Máy ảo Slave đang tiếp nhận source code từ Git..."
                sh "ls -la" // Lệnh Linux liệt kê xem code đã kéo về đủ chưa
            }
        }

        stage('Execute JMeter') {
            steps {
                script {
                    // Dùng lệnh sh của Linux để xóa log cũ và tạo thư mục mới
                    sh "rm -rf reports/* || true"
                    sh "mkdir -p reports"

                    def baseName = params.SCRIPT_FILENAME.replace('.jmx', '')
                    def reportName = "${baseName}_Build_${BUILD_NUMBER}"

                    def loadConfig = readProperties file: 'config/env.properties'
                    env.BASE_URL = loadConfig["baseUrl"]

                    // Lệnh sh gọi JMeter chạy dưới nền Linux
                    sh """
                        ${JMETER_HOME}/jmeter -n -t "scripts/${params.SCRIPT_FILENAME}" \
                        -l "reports/${reportName}.jtl" \
                        -e -o "reports/${reportName}_Dashboard" \
                        -JbaseUrl="${env.BASE_URL}"
                    """
                }
            }
        }
    }

    post {
        always {
            script {
                def baseName = params.SCRIPT_FILENAME.replace('.jmx', '')
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: "reports/${baseName}_Build_${BUILD_NUMBER}_Dashboard",
                    reportFiles: 'index.html',
                    reportName: "Báo cáo: ${baseName}"
                ])
            }
        }
    }
}
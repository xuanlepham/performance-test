pipeline {
    agent { label 'jmeter-slave' }

    options {
        timeout(time: 60, unit: 'MINUTES')
    }

    parameters {
        string(name: 'SCRIPT_FILENAME', defaultValue: 'Loadtestdemo.jmx', description: 'Tên file kịch bản trong thư mục scripts/')
        string(name: 'TARGET_URL', defaultValue: 'parabank.parasoft.com', description: 'Nhập domain server cần test')
        string(name: 'THREADS', defaultValue: '10', description: 'Số lượng Virtual Users (Number of Threads)')
        string(name: 'RAMP_UP', defaultValue: '5', description: 'Thời gian Ramp-up (giây)')
        string(name: 'LOOPS', defaultValue: '1', description: 'Số vòng lặp (Để trống hoặc -1 nếu chạy theo Duration)')
        string(name: 'DURATION', defaultValue: '300', description: 'Thời gian ngâm tải Duration bằng giây (Để trống nếu chạy theo Loop)')

        // MENU XỔ XUỐNG CHỌN CỤM SLAVE
        choice(name: 'SLAVE_CLUSTER', choices: ['none', 'cụm_1_máy', 'cụm_2_máy', 'cụm_full_3_máy'], description: 'Chọn cụm JMeter Slave (none = Chỉ chạy bằng máy hiện tại)')
    }

    environment {
        JMETER_HOME = "/home/xuanle/Desktop/dowload/apache-jmeter-5.6.3/bin"
    }

    stages {
        stage('Prepare & Checkout') {
            steps {
                echo "Máy ảo Slave đang tiếp nhận source code từ Git..."
                sh "ls -la"
            }
        }

        stage('Execute JMeter') {
            steps {
                script {
                    sh "rm -rf reports/* || true"
                    sh "mkdir -p reports"

                    def baseName = params.SCRIPT_FILENAME.replace('.jmx', '')
                    def reportName = "${baseName}_Build_${BUILD_NUMBER}"

                    // 1. Đọc file env chung để lấy baseUrl
                    def loadConfig = readProperties file: 'config/env.properties'
                    env.BASE_URL = loadConfig["baseUrl"]

                    // Xử lý giá trị an toàn cho Loop và Duration
                    def jmeterLoops = params.LOOPS ?: "1"
                    def jmeterDuration = params.DURATION ?: "0"

                    // 2. LOGIC TỰ ĐỘNG LẤY IP TỪ FILE CONFIG RIÊNG CỦA ÔNG
                    def remoteFlag = ""
                    if (params.SLAVE_CLUSTER != 'none') {
                        // Tên file lưu IP của ông là gì thì ông thay vào chữ 'slave_clusters.properties' nhé
                        def slaveConfig = readProperties file: "config/slave_clusters.properties"

                        def targetIps = slaveConfig[params.SLAVE_CLUSTER]
                        remoteFlag = "-R ${targetIps}"
                    }

                    // 3. LỆNH CHẠY CHỐT HẠ (Đã bơm đủ đồ chơi)
                    sh """
                        ${JMETER_HOME}/jmeter -n -t "scripts/${params.SCRIPT_FILENAME}" \
                        -l "reports/${reportName}.jtl" \
                        -e -o "reports/${reportName}_Dashboard" \
                        -JbaseUrl="${env.BASE_URL}" \
                        -JtargetUrl="${params.TARGET_URL}" \
                        -Jthreads="${params.THREADS}" \
                        -Jrampup="${params.RAMP_UP}" \
                        -Jduration="${jmeterDuration}" \
                        -Jloops="${jmeterLoops}" \
                        ${remoteFlag}
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
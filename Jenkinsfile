pipeline {
    agent { label 'jmeter-slave' }

    parameters {
        string(name: 'SCRIPT_FILENAME', defaultValue: 'Loadtestdemo.jmx', description: 'Tên file kịch bản trong thư mục scripts/')
        string(name: 'THREADS', defaultValue: '10', description: 'Số lượng Virtual Users (Number of Threads)')
        string(name: 'RAMP_UP', defaultValue: '5', description: 'Thời gian Ramp-up (giây)')
        string(name: 'LOOPS', defaultValue: '1', description: 'Số vòng lặp (Để trống hoặc -1 nếu chạy theo Duration)')
        string(name: 'DURATION', defaultValue: '300', description: 'Thời gian ngâm tải Duration bằng giây (Để trống nếu chạy theo Loop)')
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

                    def loadConfig = readProperties file: 'config/env.properties'
                    env.BASE_URL = loadConfig["baseUrl"]

                    // Bơm ĐẦY ĐỦ các loại biến vào lệnh. JMeter sẽ tự động phớt lờ những biến mà script không dùng tới.
                    sh """
                        ${JMETER_HOME}/jmeter -n -t "scripts/${params.SCRIPT_FILENAME}" \
                        -l "reports/${reportName}.jtl" \
                        -e -o "reports/${reportName}_Dashboard" \
                        -JbaseUrl="${env.BASE_URL}" \
                        -Jthreads="${params.THREADS}" \
                        -Jrampup="${params.RAMP_UP}" \
                        -Jloops="${params.LOOPS}" \
                        -Jduration="${params.DURATION}"
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
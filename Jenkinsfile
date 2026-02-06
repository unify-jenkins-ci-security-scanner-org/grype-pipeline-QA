pipeline {
    agent any

    environment {
        GRYPE_BINARY_DIR = "${env.WORKSPACE}/bin"
        GRYPE_SCAN_TARGET = "${env.WORKSPACE}/test-workflow-ninja"
        GRYPE_REPORT = "grype-report.sarif"
    }

    triggers {
        cron '15 04 * * 1-5' // Runs at 43:15 on every day-of-week from Monday through Friday
         }

    stages {
        stage('Install Grype') {
            steps {
                sh '''
                echo "Installing Grype..."
                mkdir -p ${GRYPE_BINARY_DIR}
                export PATH=${GRYPE_BINARY_DIR}:$PATH
                curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b ${GRYPE_BINARY_DIR}
                grype version
                '''
            }
        }
        
        stage('List Files') {
            steps {
                sh '''
                echo "📁 Current workspace contents:"
                ls -la ${WORKSPACE}
                '''
            }
        }

        stage('Scan Folder with Grype') {
            steps {
                sh '''
                echo "Scanning folder '${GRYPE_SCAN_TARGET}' with Grype..."
                ${GRYPE_BINARY_DIR}/grype ${GRYPE_SCAN_TARGET} -o sarif --file ${GRYPE_REPORT} 2>/dev/null
                '''
            }
        }

        stage('Display SARIF Report') {
            steps {
                sh '''
                echo "=== Grype SARIF Report ==="
                cat ${GRYPE_REPORT}
                '''
            }
        }
        stage('Security Scan') {
            steps {
                registerSecurityScan(
                    // Security Scan to include
                    artifacts: "${GRYPE_REPORT}",
                    format: "sarif",
                    archive: true
                )
            }
        }
    }

    // post {
    //     always {
    //                 registerSecurityScan(
    //                     artifacts: '${GRYPE_REPORT}',
    //                     format: 'sarif',
    //                     archive: true
    //                 )
    //          // archiveArtifacts artifacts: "${GRYPE_REPORT}", fingerprint: true
    //     }
    // }
}

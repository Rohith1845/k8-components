pipeline {
    // These are pre-build sections
    agent {
        node {
            label 'Agent-1'
        }
    }
    // tools{
    //     nodejs 'node20'
    //     terraform 'terraform-1.5'
    //     dockerTool 'docker-20'
    // }
    environment {
        COURSE = "Jenkins"
        appVersion = ""
        ACC_ID = "131315333865"
        project = "roboshop"
        component = "catalogue"
    }
    options {
        timeout(time: 10, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
    // This is build section
    stages {    
        stage('Read Version') {
            steps {
                script{
                    def packageJSON = readJSON file: 'package.json'
                    appVersion = packageJSON.version
                    echo "app version: ${appVersion}"
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                script{
                    sh """
                        npm install
                    """
                }
            }
        }
        stage('Unit Test') {
            steps {
                script{
                    sh """
                        npm test
                    """
                }
            }
        }
        // Here select the scanner tool and send reports to server
        // stage('Sonar Scan'){
        //     environment {
        //         def scannerHome = tool 'sonar-8.0'
        //     }
        //     steps {
        //         script{
        //             withSonarQubeEnv('sonar-server'){
        //                 sh "${scannerHome}/bin/sonar-scanner"
        //             }
        //         }
        //     }
        // }
        // stage('Quality Gate'){
        //     steps {
        //         timeout(time: 1, unit: 'HOURS'){
        //             waitForQualityGate abortPipeline:true
        //         }
        //     }
        // }
        stage('Build Image') {
            steps {
                script{
                    withAWS(region:'us-east-1',credentials:'aws-creds') {
                        sh """
                            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                            docker images
                            docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """
                    }
                }
            }
        }
        stage('Dependabot Alerts') {
            environment {
                GITHUB_TOKEN = credentials('github-token')
                GIT_OWNER = 'rohith1845'
                GITHUB_API = 'https://api.github.com'
                GITHUB_REPO = 'catalogue'
            }
            steps {
                sh '''
                curl -s \
                -H "Authorization: Bearer ${GITHUB_TOKEN}" \
                -H "Accept: application/vnd.github+json" \
                ${GITHUB_API}/repos/${GIT_OWNER}/${GITHUB_REPO}/dependabot/alerts

                echo "${response}" > dependabot_alerts.json

                    high_critical_open_count=$(echo "${response}" | jq '[.[] 
                        | select(
                            .state == "open"
                            and (.security_advisory.severity == "high"
                                or .security_advisory.severity == "critical")
                        )
                    ] | length')

                    echo "Open HIGH/CRITICAL Dependabot alerts: ${high_critical_open_count}"

                    if [ "${high_critical_open_count}" -gt 0 ]; then
                        echo "❌ Blocking pipeline due to OPEN HIGH/CRITICAL Dependabot alerts"
                        echo "Affected dependencies:"
                        echo "$response" | jq '.[] 
                        | select(.state=="open" 
                        and (.security_advisory.severity=="high" 
                        or .security_advisory.severity=="critical"))
                        | {dependency: .dependency.package.name, severity: .security_advisory.severity, advisory: .security_advisory.summary}'
                        exit 1
                    else
                        echo "✅ No OPEN HIGH/CRITICAL Dependabot alerts found"
                    fi
                '''
            }
        }
        stage('Trivy scan'){
            steps{
                script{
                    sh """
                        trivy image \
                        --scanners vuln \
                        --severity HIGH,CRITICAL,MEDIUM \
                        --pkg-types os \
                        --exit-code 1 \
                        --format table \
                        ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                    """
                }
            }
        }

    }
    post{
        always{
            echo 'I will always say Hello again!'
            cleanWs()
        }
        success {
            echo 'I will run if success'
        }
        failure {
            echo 'I will run if failure'
        }
        aborted {
            echo 'pipeline is aborted'
        }
    }
}
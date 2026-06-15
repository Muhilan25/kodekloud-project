pipeline {
    agent any
    tools {
        nodejs 'nodejs-22.6.0'
    }

    stages{
        stage("install dependencies") {
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage("Dependency scanning") {
            parallel {
                stage("NPM Dependency Audit") {
                    steps {
                        sh '''
                            npm audit --audit-level=critical
                            echo $?
                        '''
                    }
                }

                stage("OWASP Dependency check") {
                    steps {
                        dependencyCheck additionalArguments: '''
                            --scan \'./\'
                            --out  \'./\'
                            --format  \'ALL\'
                            --prettyPrint''', odcInstallation: 'OWASP-DepCheck-10'
                        dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true
                        publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir:
                         './', reportFiles: 'dependency-check-jenkins.html', reportName: 'Depedency Check HTML
                          Report', reportTitles: '', useWrapperFileDirectly: true])
                        
                    }
                }
            }
        }
    }
}
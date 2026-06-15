pipeline {
    agent any
    tools {
        nodejs 'nodejs-22.6.0'
    }
    environment {
        MONGO_URL = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
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

                        junit allowEmptyResults: true, testResults: 'dependency-check-junit.xml'

                        publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir:
                        './', reportFiles: 'dependency-check-jenkins.html', reportName: 'Depedency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        
                    }
                }
            }
        }

        stage("unit testing") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'mongo-db-cred', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                    sh 'npm test'
                }
                junit allowEmptyResults: true, testResults: 'test-results.xml'
               
            }
        }
    }
}
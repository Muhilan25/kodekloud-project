pipeline {
    agent any
    tools {
        nodejs 'nodejs-22.6.0'
    }
    environment {
        MONGO_URL = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
        SCANNER_HOME = tool('sonarqube-scanner')
    }

    stages{
        stage("install dependencies") {
            steps {
                sh 'npm install --no-audit'
            }
        }

         stage('Check Scanner') {
            steps {
                sh '''
                echo $SCANNER_HOME
                ls -la $SCANNER_HOME
                '''
            }
        }

        // stage("Dependency scanning") {
        //     parallel {
        //         stage("NPM Dependency Audit") {
        //             steps {
        //                 sh '''
        //                     npm audit --audit-level=critical
        //                     echo $?
        //                 '''
        //             }
        //         }

        //         stage("OWASP Dependency check") {
        //             steps {
        //                 dependencyCheck additionalArguments: '''
        //                     --scan \'./\'
        //                     --out  \'./\'
        //                     --format  \'ALL\'
        //                     --prettyPrint''', odcInstallation: 'OWASP-DepCheck-10'
        //                 dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true            
        //             }
        //         }
        //     }
        // }

        // stage("unit testing") {
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'mongo-db-cred', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
        //             sh 'npm test'
        //         }
        //         junit allowEmptyResults: true, testResults: 'test-results.xml'
               
        //     }
        // }

        // stage("SAST") {
        //     steps {
        //         withSonarQubeEnv('sonar') {
        //             sh '''
        //              ${SCANNER_HOME}/bin/sonar-scanner \
        //                 -Dsonar.projectName=Kodekloud-project \
        //                 -Dsonar.projectKey=Kodekloud-project \
        //                 -Dsonar.sources=. \
        //             '''
        //         }
        //     }
        // }


    }
    // post {
    //     always {
    //          junit allowEmptyResults: true, testResults: 'dependency-check-junit.xml'

    //          publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir:
    //          './', reportFiles: 'dependency-check-jenkins.html', reportName: 'Depedency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])
    //     }
    // }
}
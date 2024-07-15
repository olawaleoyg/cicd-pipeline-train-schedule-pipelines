pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('DeployToStaging') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'staging',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ],
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: '''
                                            echo "$USERPASS" | sudo -S apt-get update && 
                                            echo "$USERPASS" | sudo -S apt-get install -y unzip &&
                                            echo "$USERPASS" | sudo -S /usr/bin/systemctl stop train-schedule && 
                                            echo "$USERPASS" | sudo -S rm -rf /opt/train-schedule/* && 
                                            echo "$USERPASS" | sudo -S unzip /tmp/trainSchedule.zip -d /opt/train-schedule && 
                                            echo "$USERPASS" | sudo -S /usr/bin/systemctl start train-schedule
                                        '''
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}

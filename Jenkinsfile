pipeline {
    agent any

    tools {
        maven "M3"
        jdk "JDK17"
    }
    environment {
        DOCKERHUB_CRDENTIALS = credential('dockerCredential')
    }

    stages {
        stage('Git Clone') {
            steps {
                echo 'Git Clone'
                git url: 'https://github.com/AS-SNIPER/spring-petclinic.git', branch: 'main'
            }
            post {
                success {
                    echo 'Git Clone Success'
                }
                failure {
                    echo 'Git Clone Fail'
                }
            }
        }

        stage('Maven Build') {
            steps {
                echo 'Maven Build'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
        }

        // Docker image creation stage
        stage('Docker Image Build') {
            steps {
                echo 'Docker Image Build'
                dir("${env.WORKSPACE}") {
                    sh '''
                        docker build -t spring-petclinic:$BUILD_NUMBER .
                        docker tag spring-petclinic:$BUILD_NUMBER as-sniper/spring-petclinic:$BUILD_NUMBER
                    '''
                }
            }
        }

        //Docker images Push
        stage('Docker image Push'){
            steps{
                sh'''
                echo $DOCKERHUB_CRDENTIALS_PSW | docker login -u $DOCKERHUB_CRDENTIALS_USR -password-stdin
                docker push ms13200/spring-petclinic:latest
                '''
            }
        }

        // SSH Publish to remote server stage
        stage('SSH Publish') {
            steps {
                echo 'SSH Publish'
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'target',
                        transfers: [
                            sshTransfer(
                                cleanRemote: false, 
                                excludes: '', 
                                execCommand: ''' 
                                    fuser -k 8080/tcp  # Kill any process on port 8080
                                    export BUILD_ID=PetClinic  # Export a build ID environment variable
                                    nohup java -jar spring-petclinic-3.4.0-SNAPSHOT.jar >> nohup.out 2>&1 &  # Start app in background
                                ''',
                                execTimeout: 120000, 
                                flatten: false, 
                                makeEmptyDirs: false, 
                                noDefaultExcludes: false, 
                                patternSeparator: '[, ]+', 
                                remoteDirectory: '', 
                                remoteDirectorySDF: false, 
                                removePrefix: 'target',
                                sourceFiles: 'target/*.jar'
                            )
                        ],
                        usePromotionTimestamp: false,
                        useWorkspaceInPromotion: false,
                        verbose: false
                    )
                ])
            }
        }
    }
}


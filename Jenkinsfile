pipeline {
    options {
        timestamps()
        skipDefaultCheckout()
    }

    //Declares where the file will run, in this case node called worker1
    agent {
        node { label 'worker1'}
    }
    //re-runs file every 5 minutes
    triggers {
        pollSCM('H/5 * * * *')
    }

     parameters {
        string(name: 'BUILD_VERSION', defaultValue: '', description: '')
    }

    stages {
        stage('Build Version'){
            when { expression { BUILD_VERSION == '' } }
            steps{
                script {
                    BUILD_VERSION_GENERATED = VersionNumber(
                        versionNumberString: 'v${BUILD_YEAR, XX}.${BUILD_MONTH, XX}${BUILD_DAY, XX}.${BUILDS_TODAY}',
                        projectStartDate:    '1970-01-01',
                        skipFailedBuilds:    true)
                    currentBuild.displayName = BUILD_VERSION_GENERATED
                    env.BUILD_VERSION = BUILD_VERSION_GENERATED
                    env.BUILD = 'true'
                }
            }
        }
        stage('Checkout script') {
            steps {
                cleanWs()
                checkout scm
                echo 'checkout complete'
            }
        }
        stage('Application Build') {
            steps {
                sshagent (credentials: ['github_access']) {
                    withEnv([
                        "IMAGE_NAME=sample_application",
                        "BUILD_VERSION=" + (params.BUILD_VERSION ?: env.VERSION)
                    ]) {
                        cleanWs()
                        checkout scm
                        script {
                            // See: https://jenkins.io/doc/book/pipeline/docker/#building-containers
                            docker.build("${env.IMAGE_NAME}", "--build-arg --no-cache ./")
                    }
                }
            }
        }

        stage('Deploy to node') {
            steps {
                script {
                    docker.withRegistry('https://063208468694.dkr.ecr.us-west-1.amazonaws.com', 'ecr:us-west-1:github_access') {
                        sh 'docker push 063208468694.dkr.ecr.us-west-1.amazonaws.com/sample_application:$BUILD_NUMBER')
                        sh 'docker stop my-first-pipeline_main_web_1'
                        sh 'docker-compose -f docker-compose.dev.yml up -d --build'
                        }
                    }
                }
                echo 'Deployment complete'
        }
    }

}


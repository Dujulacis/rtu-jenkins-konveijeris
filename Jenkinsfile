def installPipDeps() {
    echo 'Installing required Python (pip) dependencies...'

    dir('python-greetings') {
        deleteDir()
    }

    bat 'git clone https://github.com/mtararujs/python-greetings.git'

    dir('python-greetings') {
        bat 'dir'
        bat 'python -m venv venv'
        bat 'venv\\Scripts\\python.exe -m pip install -r requirements.txt'
        stash name: 'python-greetings-venv', includes: 'venv/**'
    }

}

def deployTo(environmentName) {
    echo "Deploying application to ${environmentName} environment..."

    def port = ''
    if (environmentName == 'dev') {
        port = '7001'
    } else if (environmentName == 'stg') {
        port = '7002'
    } else if (environmentName == 'preprod') {
        port = '7003'
    } else if (environmentName == 'prod') {
        port = '7004'
    } else {
        error("Unknown environment: ${environmentName}")
    }

    dir("python-greetings-${environmentName}") {
        deleteDir()
    }

    bat "git clone https://github.com/mtararujs/python-greetings.git python-greetings-${environmentName}"

    dir("python-greetings-${environmentName}") {
        def venvPython = "${pwd()}\\venv\\Scripts\\python.exe"
        unstash 'python-greetings-venv'
        bat(returnStatus: true, script: "pm2 delete greetings-app-${environmentName}")
        bat "pm2 start app.py --name greetings-app-${environmentName} --interpreter \"${venvPython}\" -- --port ${port}"
        bat 'pm2 list'
    }
}

def runTestsOn(environmentName) {
    echo "Running tests on ${environmentName} environment..."

    dir("course-js-api-framework-${environmentName}") {
        deleteDir()
    }

    bat "git clone https://github.com/mtararujs/course-js-api-framework.git course-js-api-framework-${environmentName}"

    dir("course-js-api-framework-${environmentName}") {
        bat 'npm install'
        bat "npm run greetings greetings_${environmentName}"
    }
}

pipeline {
    agent any

    stages {
        stage('install-pip-deps') {
            steps {
                script {
                    installPipDeps()
                }
            }
        }

        stage('deploy-to-dev') {
            steps {
                script {
                    deployTo('dev')
                }
            }
        }

        stage('tests-on-dev') {
            steps {
                script {
                    runTestsOn('dev')
                }
            }
        }

        stage('deploy-to-stg') {
            steps {
                script {
                    deployTo('stg')
                }
            }
        }

        stage('tests-on-stg') {
            steps {
                script {
                    runTestsOn('stg')
                }
            }
        }

        stage('deploy-to-preprod') {
            steps {
                script {
                    deployTo('preprod')
                }
            }
        }

        stage('tests-on-preprod') {
            steps {
                script {
                    runTestsOn('preprod')
                }
            }
        }

        stage('deploy-to-prod') {
            steps {
                script {
                    deployTo('prod')
                }
            }
        }

        stage('tests-on-prod') {
            steps {
                script {
                    runTestsOn('prod')
                }
            }
        }
    }
}
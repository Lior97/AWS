pipeline {
    agent any
    tools {
        nodejs "node"
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    # enter app directory, because that's where package.json is located
                    dir("app") {
                        # update application version in the package.json file with one of these release types: patch, minor or major
                        # this will commit the version update
                        npm version minor

                        # read the updated version from the package.json file
                        def packageJson = readJSON file: 'package.json'
                        def version = packageJson.version

                        # set the new version as part of IMAGE_NAME
                        env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                    }

                 }
            }
        }
        stage('Run tests') {
            steps {
               script {
                    # enter app directory, because that's where package.json and tests are located
                    dir("app") {
                        # install all dependencies needed for running tests
                        sh "npm install"
                        sh "npm run test"
                    } 
               }
            }
        }
        stage('Build and Push docker image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Docker-Creden', usernameVariable: 'USER', passwordVariable: 'PWD')]){
                    sh "docker build -t liory97/jenkins:${IMAGE_NAME} ."
                    sh "echo ${PWD} | docker login -u ${USER} --password-stdin"
                    sh "docker push liory97/jenkins:${IMAGE_NAME}"
                }
            }
        }
        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'Git-Creden', usernameVariable: 'USER', passwordVariable: 'PWD')]) {
                        # git config here for the first time run
                        sh 'git config --global user.email "lioryasharzada@gmail.com"'
                        sh 'git config --global user.name "jenkins"'

                        sh "git remote set-url origin https://github.com/Lior97/Jenkins.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push '
                    }
                }
            }
        }
    }
}

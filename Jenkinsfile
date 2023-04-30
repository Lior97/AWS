pipeline {
    agent any
    tools {
        nodejs "node"
        git "git"
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    def version = packageJson.version
                        dir("AWS") {
                        npm version minor
                        def packageJson = readJSON file: 'package.json'
                        env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                    }

                 }
            }
        }
        stage('Run tests') {
            steps {
               script {
                    dir("AWS") {
                        sh "npm install"
                        sh "npm run test"
                    } 
               }
            }
        }
        stage('Build and Push docker image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'USER', passwordVariable: 'PWD')]){
                    sh "docker build -t liory97/AWS:${IMAGE_NAME} ."
                    sh "echo ${PWD} | docker login -u ${USER} --password-stdin"
                    sh "docker push liory97/AWS:${IMAGE_NAME}"
                }
            }
        }
         stage('deploy to EC2') {
            steps {
                script {
                   def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                   def ec2Instance = "ec2-user@3.78.97.237"

                   sshagent(['ec2-server-key']) {
                       sh "scp -o StrictHostKeyChecking=no server-cmds.sh ${ec2Instance}:/home/ec2-user"
                       sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                       sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                   }     
                }
            }
        }

        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'git-cred', usernameVariable: 'USER', passwordVariable: 'PWD')]) {
                        sh 'git config --global user.email "lioryasharzada@gmail.com"'
                        sh 'git config --global user.name "Lior97"'
                        sh "git remote set-url origin https://github.com/Lior97/AWS.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push HEAD:main'
                    }
                }
            }
        }
    }
}

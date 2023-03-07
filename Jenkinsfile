pipeline {
    agent { label 'lab2' }
    parameters {
        choice(name: 'choice', choices: ['dev', 'test', 'prod',"release"])
    } 
    stages {
        stage('build and push') {
            steps {
                echo 'build and push'
                script {
                    if (params.choice == "release") {
                        withCredentials([usernamePassword(credentialsId: 'lab2-dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                            sh """
                                docker login -u ${USER}  -p ${PASS}
                                docker build -t ${USER}/lab2-bakehouse:${BUILD_NUMBER} .
                                docker push ${USER}/lab2-bakehouse:${BUILD_NUMBER}
                                echo ${BUILD_NUMBER} > ../bakehouse-build-number.txt
                            """
                        }
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                echo 'deploy'
                script {
                    if (params.choice == "dev" || params.choice == "test"  || params.choice == "prod"  ) {
                        withCredentials([file(credentialsId: 'lab2-kubeconfig', variable: 'CONFIG')]) {
                            sh """
                                export BUILD_NUMBER=\$(cat ../bakehouse-build-number.txt)
                                mv Deployment/deploy.yaml Deployment/deploy.yaml.tmp
                                cat Deployment/deploy.yaml.tmp | envsubst > Deployment/deploy.yaml
                                rm -f Deployment/deploy.yaml.tmp
                                kubectl apply -f Deployment --kubeconfig ${CONFIG}
                            """
                        }
                    }
                }
            }
        }
    }
}

pipeline {
    environment {
    registry = "muzammilkazmi86/udac"
    registryCredential = 'dockerhub'
}
    agent any

    stages {
            stage('HTML File Lint check') {
                steps {

                    sh 'tidy -q -e *.html'



                }
        }
            stage('Dockerfile Lint check') {
                    steps {

                        sh 'hadolint --ignore DL3006 --ignore SC2028 --ignore DL3008 --ignore DL3015 Dockerfile'



                    }
        }
            stage('Docker Image Building') {
                    steps {

                        withCredentials([usernamePassword( credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'echo "the value of username is ${USERNAME} and password id ${PASSWORD}"'
                        sh "docker login -u ${USERNAME} -p ${PASSWORD}"
                        sh 'docker build --tag=muzammilkazmi86/udac:Capstone .'



                    }
        }
        }
            stage('Docker Image Push') {
                    steps {

                        sh 'docker push muzammilkazmi86/udac:Capstone'



                    }
        }
            stage('Deploy latest on backup cluster') {
                    steps {

                        sh "kubectl config use-context arn:aws:eks:us-west-2:595702470973:cluster/backup"
                        sh "kubectl apply -f ./blue.json"



                    }
        }
            stage('Approval to route traffic to backup') {
                    steps {
                        input "Does the new version looks good?"
                    }
        }
             stage('Routing traffic to backup') {
                    steps {
                        sh "kubectl apply -f ./blue-service.json"
                        sh '''ELB="$(kubectl get svc blue -o=jsonpath='{.status.loadBalancer.ingress[0].hostname}')"
                              aws route53 change-resource-record-sets --hosted-zone-id ZDQ7GFDSQJGM6 --change-batch '{ "Comment": "creating a record set",  "Changes": [ { "Action": "UPSERT", "ResourceRecordSet": { "Name": "capstone.udac.com", "Type": "CNAME", "TTL": 120, "ResourceRecords": [ { "Value": "'"$ELB"'" } ] } } ] }'
                            '''
                        
                    }
        }
            stage('Deploy latest on production cluster') {
                    steps {
                        sh "kubectl config use-context arn:aws:eks:us-west-2:595702470973:cluster/prod"
                        sh "kubectl apply -f ./green.json"



                    }
        }
            stage('Approval to route traffic to prod') {
                    steps {
                        input "Does the new version on prod looks good?"
                    }
        }
            stage('Routing traffic to prod') {
                    steps {
                        sh "kubectl apply -f ./green-service.json"
                        sh '''ELB="$(kubectl get svc green -o=jsonpath='{.status.loadBalancer.ingress[0].hostname}')"
                              aws route53 change-resource-record-sets --hosted-zone-id ZDQ7GFDSQJGM6 --change-batch '{ "Comment": "creating a record set",  "Changes": [ { "Action": "UPSERT", "ResourceRecordSet": { "Name": "capstone.udac.com", "Type": "CNAME", "TTL": 120, "ResourceRecords": [ { "Value": "'"$ELB"'" } ] } } ] }'
                            '''
                        
                    }
        }

    }
}

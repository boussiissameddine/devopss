def commit_id
pipeline {
    agent any
    stages {
        stage('preparation') {
            steps {
                checkout scm
                sh "git rev-parse --short HEAD > .git/commit-id"
                script {
                    commit_id = readFile('.git/commit-id').trim()
                }
            }
        }
        stage ('code quality') {
            steps {
                echo 'testing code quality'
               sh "mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=position-similator \
                    -Dsonar.host.url=http://localhost:9000 \
                    -Dsonar.login=c267b831e4ecea5ff75b1aa7f999a0054394a97d"
                echo 'code quality test complete'
            }
        }
        stage ('build') {
            steps {
                echo 'building maven workload'
                sh "mvn clean install"
                echo 'build complete'
            }
        }

        stage ("image build") {
            steps {
                echo 'building docker image'
                sh "docker login -u issambits -p B8lSpt=G docker.io"
                sh "docker build -t issambits/position-simulator:${commit_id} ."
                echo 'docker image built'
            }
        }
        stage ("image push") {
            steps {
                echo 'pushing docker image'
                sh "docker push issambits/position-simulator:${commit_id} "
                echo 'docker image pushed'
            }
        }
        stage('deploy') {
            steps {
                sh "sed -i -r 's|richardchesterwood/k8s-fleetman-position-simulator:release2|issambits/position-simulator:${commit_id}|' workloads.yaml"
                sh "kubectl apply -f workloads.yaml"
            }
        }
    }
}

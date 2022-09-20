pipeline {
    agent any

     parameters {
        booleanParam(name: 'Promote', defaultValue: true, description: 'Promote statement')
        string(name: 'Version', defaultValue: '1.0.0', description: 'Version')
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('main')
    }

    stages {
        stage('Build') {
            steps {
                echo 'building node-red'
				dir("${params.Build_files}") {
                    sh 'docker build . -f dockerbuild -t node-red_build'
				}
                
            }
        }
        
        stage('Tests') {
            steps {
                echo 'testing node-red'
				dir("${params.Build_files}") {
                    sh 'docker build . -f dockertest -t node-red_test'
				}
            }
        }
        
         
       stage('Deploy') {
            steps {
                echo 'deploying node-red'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker tag node-red_build:latest mihaf70/node-red-devops-deploy'
                sh 'docker push mihaf70/node-red-devops-deploy'
            }
        }

         stage('Publish'){
            when {
                expression {return params.Promote}
		    }
            steps {
                echo 'Publish'
                dir("${params.Build_files}") {
                    sh 'docker build . -f dockerpublish -t node-red_publish'
				}
                sh "docker run --volume /var/jenkins_home/workspace/node-red-pipeline:/finalArchive node-red_publish mv archive.tar.xz /finalArchive"
                 dir('/var/jenkins_home/workspace/node-red-pipeline'){
                    sh "mv archive.tar.xz archive-${params.Version}.tar.xz"
			        archiveArtifacts artifacts: "archive-${params.Version}.tar.xz"
			    }
            }
        }
	 }

	post {
        success {
            echo 'Pipeline succsess'
        }
        failure {
            echo 'Pipeline fail'
        }
    }
}

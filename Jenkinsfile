
import java.time.*
import java.time.format.DateTimeFormatter

def now = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMddHHmmss"))


pipeline {

    agent {
        kubernetes {
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    job: build-service
spec:
  containers:
  - name: liquibase
    image: liquibase/liquibase:4.3
    command:
      - liquibase
      - --defaultsFile=/config/liquibase.properties
      - --changeLogFile=/config/changelog.sql
      - update
    volumeMounts:
      - name: liquibase-config
        mountPath: /config
  - name: docker
    image: docker:18.09.2
    command: ["cat"]
    tty: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
  volumes:
  - name: liquibase-config
    configMap:
      name: liquibase-config
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
"""
        }
}

    stages {
        stage ('checkout') {
            steps {
                script {
                    def repo = checkout scm
                    revision = sh(script: 'git log -1 --format=\'%h.%ad\' --date=format:%Y%m%d-%H%M | cat', returnStdout: true).trim()
                    branch = repo.GIT_BRANCH.take(20).replaceAll('/', '_')
                    if (branch != 'prod') {
                        revision += "-${branch}"
                    }
                    sh "echo 'Building revision: ${revision}'"
                }
            }

        }
    stage ('deploy to dev') {
        when {
            expression {
                branch == 'dev'
            }
        }
        steps {
            println "${branch}"

        }
    }

        stage ('deploy to test') {
            when {
                expression {
                    branch == 'main'
                }
            }
            steps {
                println "deploying code from ${branch} branch to testing env"

            }
        }
        stage ('perform testing ') {
            when {
                expression {
                    branch == 'main'
                }
            }
            steps {
                println "testing code from ${branch} branch"

            }
        }

        stage ('deploy on prod') {
            when {
                expression {
                    branch == 'main'
                }
            }
            steps {
                println "DEPLOYING CODE FROM  ${branch} branch to prod env"
                script {
                    
                    sh "ls -alh"
                }
            }
        }
        
    }
}
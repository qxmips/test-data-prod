
import java.time.*
import java.time.format.DateTimeFormatter

def now = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMddHHmmss"))


pipeline {

    agent {
        kubernetes {
//            defaultContainer 'jnlp'
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
      - "bash"
      - "-c"
      - |
        while true; do /bin/sleep 10; done;
    volumeMounts:
      - name: liquibase-config
        mountPath: /config
  volumes:
  - name: liquibase-config
    configMap:
      name: liquibase-config
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
            container('liquibase'){
                script {
                    
                    sh "ls -alh"
                }

            }


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
                script {
                    
                    sh "liquibase --classpath=/config/ --defaultsFile=/config/liquibase.properties --changeLogFile=changelog.sql update"
                }

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
                    
                    sh "liquibase --classpath=/config/ --defaultsFile=/config/liquibase.properties --changeLogFile=changelog.sql update"
                }
            }
        }
        
    }
}
def version, mvnCmd = "mvn -s configuration/cicd-settings-nexus3.xml"
def DEV_PROJECT = 'demodev'
def SIT_PROJECT = 'demosit'
def UAT_PROJECT = 'demouat'
def PROD_PROJECT = 'demoprod'
pipeline {
      agent {
              label 'docker-jnlp-slave'
            }
            stages {
              stage('Build App') {
                steps {
                  //git branch: 'master', url: 'https://github.com/intachompoo/openshift-tasks-ex.git'
                  //script {
                      //def pom = readMavenPom file: 'pom.xml'
                      //version = pom.version
                  //}
                  sh "${mvnCmd} install -DskipTests=true"
                  //sh "mvn install -DskipTests=true"
                }
              }
              stage('Test') {
                steps {
                  //sh "${mvnCmd} test"
                  //step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
                  echo "SUCESS"
                }
              }
              stage('Code Analysis') {
                steps {
                  script {
                    sh "${mvnCmd} sonar:sonar -Dsonar.host.url=http://sonarqube-demotools.apps.180.222.156.219.xip.io -DskipTests=true"
                  }
                }
              }
              stage('Archive App') {
                steps {
                  sh "${mvnCmd} deploy -DskipTests=true -P nexus3"
                }
              }
              stage('Create Image Builder') {
                when {
                  expression {
                    openshift.withCluster( 'myCluster' ) {
                      openshift.withProject($DEV_PROJECT) {
                        return !openshift.selector("bc", "tasks").exists();
                      }
                    }
                  }
                }
                steps {
                  script {
                    openshift.withCluster( 'myCluster' ) {
                      openshift.withProject($DEV_PROJECT) {
                        openshift.newBuild("--name=tasks", "--image-stream=jboss-eap70-openshift:1.5", "--binary=true")
                      }
                    }
                  }
                }
              }
              stage('Build Image') {
                steps {
                  sh "rm -rf oc-build && mkdir -p oc-build/deployments"
                  sh "cp target/openshift-tasks.war oc-build/deployments/ROOT.war"

                  script {
                    openshift.withCluster( 'myCluster' ) {
                      openshift.withProject($DEV_PROJECT) {
                        openshift.selector("bc", "tasks").startBuild("--from-dir=oc-build", "--wait=true")
                      }
                    }
                  }
                }
              }
              stage('Create DEV') {
                when {
                  expression {
                    openshift.withCluster( 'myCluster' ) {
                      openshift.withProject($DEV_PROJECT) {
                        return !openshift.selector('dc', 'tasks').exists()
                      }
                    }
                  }
                }
                steps {
                  script {
                    openshift.withCluster( 'myCluster' ) {
                      openshift.withProject($DEV_PROJECT) {
                        def app = openshift.newApp("tasks:latest")
                        app.narrow("svc").expose();

                        openshift.set("probe dc/tasks --readiness --get-url=http://:8080/ws/demo/healthcheck --initial-delay-seconds=30 --failure-threshold=10 --period-seconds=10")
                        openshift.set("probe dc/tasks --liveness  --get-url=http://:8080/ws/demo/healthcheck --initial-delay-seconds=180 --failure-threshold=10 --period-seconds=10")

                        def dc = openshift.selector("dc", "tasks")
                        while (dc.object().spec.replicas != dc.object().status.availableReplicas) {
                            sleep 10
                        }
                        openshift.set("triggers", "dc/tasks", "--manual")
                      }
                    }
                  }
                }
              }
              stage('Deploy DEV') {
                steps {
                  script {
                    openshift.withCluster( 'myCluster' ) {
                      openshift.withProject($DEV_PROJECT) {
                        openshift.selector("dc", "tasks").rollout().latest();
                      }
                    }
                  }
                }
              }
              stage('Create SIT Image') {
                steps {
                  script {
                    openshift.withCluster( 'myCluster' ) {
                      openshift.tag("${$DEV_PROJECT}/tasks:latest", "${$SIT_PROJECT}/tasks:latest")
                    }
                  }
                }
              }
              stage('Deploy SIT') {
                steps {
                  script {
                    openshift.withCluster( 'myCluster' ) {
                      openshift.withProject($SIT_PROJECT) {
                        if (openshift.selector('dc', 'tasks').exists()) {
                          openshift.selector('dc', 'tasks').delete()
                          openshift.selector('svc', 'tasks').delete()
                          openshift.selector('route', 'tasks').delete()
                        }
                        openshift.newApp("tasks:latest").narrow("svc").expose()
                        openshift.set("probe dc/tasks --readiness --get-url=http://:8080/ws/demo/healthcheck --initial-delay-seconds=30 --failure-threshold=10 --period-seconds=10")
                        openshift.set("probe dc/tasks --liveness  --get-url=http://:8080/ws/demo/healthcheck --initial-delay-seconds=180 --failure-threshold=10 --period-seconds=10")
                        def dc = openshift.selector("dc", "tasks")
                        while (dc.object().spec.replicas != dc.object().status.availableReplicas) {
                            sleep 10
                        }
                        openshift.set("triggers", "dc/tasks", "--manual")
                      }
                    }
                  }
                }
              }
              stage('Create UAT Image') {
                steps {
                  script {
                    openshift.withCluster( 'myCluster' ) {
                      openshift.tag("${$SIT_PROJECT}/tasks:latest", "${$UAT_PROJECT}/tasks:latest")
                    }
                  }
                }
              }
              stage('Deploy UAT') {
                steps {
                  script {
                    openshift.withCluster( 'myCluster' ) {
                      openshift.withProject($UAT_PROJECT) {
                        if (openshift.selector('dc', 'tasks').exists()) {
                          openshift.selector('dc', 'tasks').delete()
                          openshift.selector('svc', 'tasks').delete()
                          openshift.selector('route', 'tasks').delete()
                        }
                        openshift.newApp("tasks:latest").narrow("svc").expose()
                        openshift.set("probe dc/tasks --readiness --get-url=http://:8080/ws/demo/healthcheck --initial-delay-seconds=30 --failure-threshold=10 --period-seconds=10")
                        openshift.set("probe dc/tasks --liveness  --get-url=http://:8080/ws/demo/healthcheck --initial-delay-seconds=180 --failure-threshold=10 --period-seconds=10")
                        def dc = openshift.selector("dc", "tasks")
                        while (dc.object().spec.replicas != dc.object().status.availableReplicas) {
                            sleep 10
                        }
                        openshift.set("triggers", "dc/tasks", "--manual")
                      }
                    }
                  }
                }
              }
              stage('Promote to PROD?') {
                steps {
                  timeout(time:15, unit:'MINUTES') {
                      input message: "Promote to PROD?", ok: "Promote"
                  }

                  script {
                    openshift.withCluster( 'myCluster' ) {
                      openshift.tag("${$DEV_PROJECT}/tasks:latest", "${$PROD_PROJECT}/tasks:${version}")
                    }
                  }
                }
              }
              stage('Deploy PROD') {
                steps {
                  script {
                    openshift.withCluster( 'myCluster' ) {
                      openshift.withProject($PROD_PROJECT) {
                        if (openshift.selector('dc', 'tasks').exists()) {
                          openshift.selector('dc', 'tasks').delete()
                          openshift.selector('svc', 'tasks').delete()
                          openshift.selector('route', 'tasks').delete()
                        }

                        openshift.newApp("tasks:${version}").narrow("svc").expose()
                        openshift.set("probe dc/tasks --readiness --get-url=http://:8080/ws/demo/healthcheck --initial-delay-seconds=30 --failure-threshold=10 --period-seconds=10")
                        openshift.set("probe dc/tasks --liveness  --get-url=http://:8080/ws/demo/healthcheck --initial-delay-seconds=180 --failure-threshold=10 --period-seconds=10")
                      }
                    }
                }
            }
        }
    }
}

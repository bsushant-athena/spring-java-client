def pushContainerImage(srcImage, destImageName) {
    
    stage('Tag and Push Docker Image') {

        def ARTIFACTORY_BASE_URL = 'docker.artifactory.aws.athenahealth.com';
        def imageName = ARTIFACTORY_BASE_URL + '/' + destImageName
        sh 'docker tag ' + srcImage + ' ' + imageName
        sh 'docker push ' + imageName
    }
}

node {

    def buildNumber = "${BUILD_NUMBER}"
    def tag = currentBuild.number

    //this version is only for gateway-client and gateway-client-spring-boot-2
    def snapshot_version = '1.0.1'

    def maven_repo = 'libs-snapshot-local'
    def mvnImage = docker.image('maven:3.3.9-jdk-8')
    def artifactory_address = 'https://artifactory.aws.athenahealth.com:443/'
    def dockerLogLevel = '-l warn'
    def pipeline = new cicd.Pipeline()
    def is_release_ready = true
    def branch_name

    println "Build number: " + buildNumber

    stage('Preparation') {
        pipeline.cleanupAndCheckout()
        branch_name =  env.BRANCH_NAME.replace("/","-").toLowerCase()
    }

    if((env.BRANCH_NAME.startsWith("release/") || env.BRANCH_NAME == 'master') && is_release_ready == true) {
        stage('Build and Publish') {
             snapshot_version = snapshot_version + '.RELEASE'
             dir('gateway-client'){
                 maven_repo = 'libs-release-local'
                 //CONFIG FILE PLUGIN CONTAINS THE SETTINGS.XML
                 pipeline.writeMavenSettings([credId:"SVC-RPRCI-AD",fileName: "global_settings.xml"]);
                 mvnImage.inside(dockerLogLevel) {
                     sh "mvn -B -gs global_settings.xml -P deployment-aws -s settings.xml -Dmaven.wagon.http.ssl.insecure=true clean install -Dversion=${snapshot_version}"
                     sh "mvn -B -gs global_settings.xml -P deployment-aws -s settings.xml deploy -Dmaven.wagon.http.ssl.insecure=true -DpomFile=pom.xml -Dversion=${snapshot_version}"
                 }
             }
             dir('gateway-client-spring-boot-2'){
                 maven_repo = 'libs-release-local'
                 //CONFIG FILE PLUGIN CONTAINS THE SETTINGS.XML
                 pipeline.writeMavenSettings([credId:"SVC-RPRCI-AD",fileName: "global_settings.xml"]);
                 mvnImage.inside(dockerLogLevel) {
                     sh "mvn -B -gs global_settings.xml -P deployment-aws -s settings.xml -Dmaven.wagon.http.ssl.insecure=true clean install -Dversion=${snapshot_version}"
                     sh "mvn -B -gs global_settings.xml -P deployment-aws -s settings.xml deploy -Dmaven.wagon.http.ssl.insecure=true -DpomFile=pom.xml -Dversion=${snapshot_version}"
                 }
             }
             //dir('eureka-client'){
             //     snapshot_version = '0.0.1'
             //     snapshot_version = snapshot_version + '.RELEASE'
             //     maven_repo = 'libs-release-local'
             //     //CONFIG FILE PLUGIN CONTAINS THE SETTINGS.XML
             //     pipeline.writeMavenSettings([credId:"SVC-RPRCI-AD",fileName: "global_settings.xml"]);
             //     mvnImage.inside(dockerLogLevel) {
             //         sh "mvn -B -gs global_settings.xml -P deployment-aws -s settings.xml -Dmaven.wagon.http.ssl.insecure=true clean install -Dversion=${snapshot_version}"
             //         sh "mvn -B -gs global_settings.xml -P deployment-aws -s settings.xml -Dmaven.wagon.http.ssl.insecure=true deploy -DpomFile=pom.xml -Dversion=${snapshot_version}"
             //     }
             //}
        }
    }
    //publishing new one(gateway-client) to the artifactory
    if(env.BRANCH_NAME == 'develop') {
        stage('Build and Publish') {
             dir('gateway-client'){
                 //CONFIG FILE PLUGIN CONTAINS THE SETTINGS.XML
                 pipeline.writeMavenSettings([credId:"SVC-RPRCI-AD",fileName: "global_settings.xml"]);
                 mvnImage.inside(dockerLogLevel) {
                     snapshot_version = snapshot_version + '-SNAPSHOT'
                     sh "mvn -B -gs global_settings.xml -P deployment-aws -s settings.xml -Dmaven.wagon.http.ssl.insecure=true clean install -Dversion=${snapshot_version}"
                     sh "mvn -B -gs global_settings.xml -P deployment-aws -s settings.xml deploy -DpomFile=pom.xml -Dmaven.wagon.http.ssl.insecure=true -Dversion=${snapshot_version}"
                 }
             }
             dir('gateway-client-spring-boot-2'){
                 //CONFIG FILE PLUGIN CONTAINS THE SETTINGS.XML
                 pipeline.writeMavenSettings([credId:"SVC-RPRCI-AD",fileName: "global_settings.xml"]);
                 mvnImage.inside(dockerLogLevel) {
                     snapshot_version = snapshot_version + '-SNAPSHOT'
                     sh "mvn -B -gs global_settings.xml -P deployment-aws -s settings.xml -Dmaven.wagon.http.ssl.insecure=true clean install -Dversion=${snapshot_version}"
                     sh "mvn -B -gs global_settings.xml -P deployment-aws -s settings.xml deploy -DpomFile=pom.xml -Dmaven.wagon.http.ssl.insecure=true -Dversion=${snapshot_version}"
                 }
             }
        }
    }

    //publishing java-sample-app docker image to the artifactory
    if(env.BRANCH_NAME.startsWith("feature/") || env.BRANCH_NAME.startsWith("release/") || env.BRANCH_NAME == 'master'|| env.BRANCH_NAME == 'develop') {
        stage('building jar') {
            dir('gateway-client-sample'){
                pipeline.writeMavenSettings([credId:"SVC-RPRCI-AD",fileName: "global_settings.xml"]);
                mvnImage.inside(dockerLogLevel) {
                    sh "mvn -B -gs global_settings.xml -P deployment-aws -s settings.xml -Dmaven.wagon.http.ssl.insecure=true clean install"
                    //To extract Jar File
                    sh "find . -name '*.jar' &&  jar -tvf ./target/java-gateway-client-sample-0.0.2-SNAPSHOT-spring-boot.jar"
                    //To display table of content
                    sh "jar -xvf ./target/java-gateway-client-sample-0.0.2-SNAPSHOT-spring-boot.jar && find . -name '*.jar'"
                }
            }
            dir('gateway-client-sample-spring-boot-2'){
                pipeline.writeMavenSettings([credId:"SVC-RPRCI-AD",fileName: "global_settings.xml"]);
                mvnImage.inside(dockerLogLevel) {
                    sh "mvn -B -gs global_settings.xml -P deployment-aws -s settings.xml -Dmaven.wagon.http.ssl.insecure=true clean install"
                    //To extract Jar File
                    sh "find . -name '*.jar' &&  jar -tvf ./target/java-gateway-client-sample-2-0.0.2-SNAPSHOT-spring-boot-2.jar"
                    //To display table of content
                    sh "jar -xvf ./target/java-gateway-client-sample-2-0.0.2-SNAPSHOT-spring-boot-2.jar && find . -name '*.jar'"
                }
            }
        }
        stage('Java Client') {
            dir('gateway-client-sample/docker'){
                def javaclientsampleImage = pipeline.buildDockerImage(
                        appName: 'java_demo_client',
                        appVersion: snapshot_version+"-"+branch_name,
                        additionalBuildParams: '-f Dockerfile .'
                )
                pipeline.pushContainerImage(image: javaclientsampleImage)
                pushContainerImage(javaclientsampleImage.imageName(), 'java_demo_client:' + snapshot_version+"-"+branch_name)
            }
            dir('gateway-client-sample-spring-boot-2/docker'){
                def javaclientsampleImage = pipeline.buildDockerImage(
                        appName: 'java_demo_client_springboot2',
                        appVersion: snapshot_version+"-"+branch_name,
                        additionalBuildParams: '-f Dockerfile .'
                )
                pipeline.pushContainerImage(image: javaclientsampleImage)
                pushContainerImage(javaclientsampleImage.imageName(), 'java_demo_client:' + snapshot_version+"-"+branch_name)
            }
        }
    }
}

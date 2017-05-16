fhBuildNode(['label': 'java-ubuntu']) {

    stage('Setup') {
        if (env.NEXUS_SERVER_ENABLED) {
            def m2Config =
                """<?xml version="1.0" encoding="UTF-8"?>
                    <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation= "http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
                    <mirrors>
                        <mirror>
                            <id>nexus</id>
                            <mirrorOf>central</mirrorOf>
                            <name>Nexus Mirror</name>
                            <url>${env.NEXUS_SERVER_URL}/repository/maven-all-public/</url>
                        </mirror>
                    </mirrors>
           </settings>"""
            writeFile file: "m2settings.xml", text: m2Config
            env.M2_SETTINGS = 'm2settings.xml'
            print "NEXUS SERVER ENABLED: ${env.NEXUS_SERVER_URL}"
        } else {
            env.M2_SETTINGS = '~/.m2/settings.xml'
            print "NEXUS SERVER NOT ENABLED"
        }
    }

    stage('Build') {
        sh "mvn -s ${env.M2_SETTINGS} clean verify -Ptest,dist -Dmaven.test.failure.ignore=false -Dups.ddl_value=update -DskipTests"
        def version = sh(returnStdout: true, script: "mvn help:evaluate -Dexpression=project.version | grep -v \"^\\[\" | tail -1 | cut -f1 -d\"-\"").trim()

        print version

        sh "cp servers/auth-server/target/auth-server.war ./unifiedpush-auth-server-${version}-${env.BUILD_NUMBER}.war"
        sh "cp servers/ups-as7/target/ag-push.war ./unifiedpush-server-as7-${version}-${env.BUILD_NUMBER}.war"

        writeFile file: "VERSION.txt", text: "${version}-${env.BUILD_NUMBER}"

        archiveArtifacts "unifiedpush-auth-server-*.war, unifiedpush-server-as7-*.war, dist/target/*.tar.gz, VERSION.txt"
    }

}

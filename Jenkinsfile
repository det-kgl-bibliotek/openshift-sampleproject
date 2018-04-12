import java.security.MessageDigest

String generateMD5_A(String s) {
    return MessageDigest.getInstance("MD5").digest(s.bytes).encodeHex().toString()
}



openshift.withCluster() { // Use "default" cluster or fallback to OpenShift cluster detection
    echo "Hello from the project running Jenkins: ${openshift.project()}"

    stage('environment') {
        sh 'env > env.txt'
        for (String i : readFile('env.txt').split("\r?\n")) {
            println i
        }

    }


    stage('Deploy test') {

        //TODO jenkins must have right on openshift project for this to work
        //https://docs.openshift.com/container-platform/3.7/admin_solutions/user_role_mgmt.html#share-templates-cluster

        //oc policy add-role-to-user registry-viewer standalone-jenkins/jenkins
        Object projectName = encodeName("${JOB_NAME}")

        echo "name="
        print projectName

        openshift.verbose()
        // Get details printed to the Jenkins console and pass high --log-level to all oc commands

        try {
            openshift.newProject(projectName)
        } catch (e) {

            //            oc delete project sampleproject-other-branch-postgres
            openshift.selector("project/${projectName}").delete()

            sh "until ! oc get project ${projectName}; do date;sleep 2; done"

        }

        openshift.verbose(false) // Turn it back


        openshift.withProject(projectName) {

            openshift.selector('all', [from: projectName]).delete()
            openshift.selector('secrets', [from: projectName]).delete()

            def created = openshift.newApp(
                    '--template=postgresql-ephemeral',
                    "--name='" + projectName + "'",
                    "--labels=from='" + projectName + "'")

            echo "new-app created ${created.count()} objects named: ${created.names()}"

            created.describe()
        }

//            for ( obj in created ) {
//                obj.metadata.labels[ "build" ] = JOB_NAME
//            }

//            JOB_NAME=det-kgl-bibliotek/openshift-jenkins-attempt/master
    }


    node('maven') {
//        stage('checkout') {
//            checkout scm
//        }
//
//        stage('Build') {
//            sh "mvn clean package"
//        }

    }
}

private Object encodeName(groovy.lang.GString jobName) {
    def name = jobName
            .replaceAll("\\s", "-")
            .replaceAll("_", "-")
            .replaceFirst("^[^/]+/", '')
            .replace("/", '-')
            .replaceAll("^openshift-", "")
    return name
}
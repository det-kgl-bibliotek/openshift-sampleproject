import java.security.MessageDigest

String generateMD5_A(String s){
    return MessageDigest.getInstance("MD5").digest(s.bytes).encodeHex().toString()
}


openshift.withCluster() { // Use "default" cluster or fallback to OpenShift cluster detection
    echo "Hello from the project running Jenkins: ${openshift.project()}"

    def projectName = BRANCH_NAME

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
        def name = "${JOB_NAME}-postgres"
                .replaceAll("\\s","-")
                .replaceFirst("^[^/]+/", '')
                .replace("/", '-');
        echo "name="
        print name

        def labels_from = "${JOB_NAME}"
                .replaceAll("\\s","-")
                .replaceFirst("^[^/]+/", '')
                .replace("/", '-');
        echo "labels_from="
        print labels_from


        openshift.verbose()
        // Get details printed to the Jenkins console and pass high --log-level to all oc commands

        try {
            openshift.newProject(name)
        } catch (e){

        }

        openshift.withProject(name) {

            openshift.selector('all', [from: labels_from]).delete()
            openshift.selector('secrets', [from: labels_from]).delete()

            def created = openshift.newApp(
                    '--template=postgresql-ephemeral',
                    "--name='" + name + "'",
                    "--labels=from='" + labels_from + "'")

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

openshift.withCluster() { // Use "default" cluster or fallback to OpenShift cluster detection
    echo "Hello from the project running Jenkins: ${openshift.project()}"


    //ALL THIS RUNS ON THE JENKINS NODE

    //Print environment, for debug purposes
    stage('environment') {
        sh 'env > env.txt'
        for (String i : readFile('env.txt').split("\r?\n")) {
            println i
        }
    }

    stage('Deploy test') {

        String projectName = encodeName("${JOB_NAME}")
        echo "name=${projectName}"

        recreateProject(projectName)


        openshift.withProject(projectName) {

            //Delete stuff already there (unnessesary if the project have been recreated, but kept here as inspiration)
            openshift.selector('all', [from: projectName]).delete()
            openshift.selector('secrets', [from: projectName]).delete()

            //Create a new Postgres instance
            def created = openshift.newApp(
                    '--template=postgresql-ephemeral',
                    "--name='" + projectName + "'",
                    "--labels=from='" + projectName + "'")

            echo "new-app created ${created.count()} objects named: ${created.names()}"

            created.describe()
        }
    }


    //GO TO A MAVEN NODE
    node('maven') {
        stage('checkout') {
            checkout scm
        }

        stage('Build') {
            sh "mvn clean package"
        }
    }
}

private void recreateProject(String projectName) {
    //Delete the project, ignore errors if the project does not exist
    try {
        openshift.selector("project/${projectName}").delete()
    } catch (e) {

    }

    //Wait for the project to be gone
    sh "until ! oc get project ${projectName}; do date;sleep 2; done; exit 0"

    //Create the project
    openshift.newProject(projectName)
}

/**
 * Encode the jobname as a valid openshift project name
 * @param jobName the name of the job
 * @return the jobname as a valid openshift project name
 */
private String encodeName(groovy.lang.GString jobName) {
    def name = jobName
            .replaceAll("\\s", "-")
            .replaceAll("_", "-")
            .replaceFirst("^[^/]+/", '')
            .replace("/", '-')
            .replaceAll("^openshift-", "")
    return name
}
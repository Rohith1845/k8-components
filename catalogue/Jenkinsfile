@Library('Jenkins-shared-library') _

def configMap = [
    project: "roboshop",
    component: "catalogue"
]

// if branch is not equal to branch then execute CI
if ( ! env.BRANCH_NAME.equalsIgnoreCase('main') ) {
    nodeJSEKSPipeline(configMap)
}
else {
    echo "please follow the cr process"
}


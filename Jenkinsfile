pipeline {
    agent any
    environment {
        PROD_SCHEMA   = "INVENTORY"
        OCI_ADB_NAME  = "JENKINSDB"
        OCI_ADB_CREDS = credentials("${OCI_ADB_NAME}_ADMIN")
    }
    stages {
        stage('main-branch') {
            when { 
                allOf { 
                    branch 'main' 
                    not { changeRequest target: 'main' } 
                }
            }
            steps {
                echo "Updating Schema ${PROD_SCHEMA}"
                sh('cd liquibase && ./cicd.py deploy --dbName $OCI_ADB_NAME --dbPass $OCI_ADB_CREDS_PSW --dbUser $PROD_SCHEMA')
           }
        } 
        stage('feature-branch') {
            when { 
                allOf {
                    not { branch 'main' } 
                    not { changeRequest() }
                }
            }
            environment {
                MYBRANCH = "${GIT_BRANCH.split("-")[0]}"
            }
            steps {
                echo "Creating Isolated Schema ${PROD_SCHEMA}${MYBRANCH}"
                sh('cd liquibase && ./cicd.py deploy --dbName $OCI_ADB_NAME --dbPass $OCI_ADB_CREDS_PSW --dbUser $PROD_SCHEMA$MYBRANCH')
           }
        }
        stage('pull-request') {
            when { changeRequest target: 'main' }
            environment {
                MYBRANCH = "${CHANGE_BRANCH.split("-")[0]}"
            }
            steps {
                echo "Testing Load of Changes into ${PROD_SCHEMA}${MYBRANCH}"
                sh('cd liquibase && ./cicd.py deploy --dbName $OCI_ADB_NAME --dbPass $OCI_ADB_CREDS_PSW --dbUser $PROD_SCHEMA$MYBRANCH')
                echo "Deleting Isolated Schema ${PROD_SCHEMA}${MYBRANCH}"
                sh('cd liquibase && ./cicd.py destroy --dbName $OCI_ADB_NAME --dbPass $OCI_ADB_CREDS_PSW --dbUser $PROD_SCHEMA$MYBRANCH')
           }
        }
    }
}
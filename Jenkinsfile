pipeline{
    agent{
        label "azure-image"
    }

    environment {
        LIC_FILE = "NOTICE.tsv"
        STREAMFLAKE_DEPEND_TAR = "sdc-dependencies.tar.gz"
        STREAMFLAKE_TARFILE = "streamsets-streamflake-all_${SPARK_ENGINE_VER}-${STREAMFLAKE_VERSION}.tgz"
        GCP_STREAMFLAKE_IMAGE_NAME = ""
        LOG_FILE = "${CLD_PROVIDER}.log"
        NETWORK_LOG_FILE = "${CLD_PROVIDER}_Network.log"
    }

    parameters {

        booleanParam(name: 'RELOAD_JOB', defaultValue: false, description: 'Reload job with latest jenkinsfile changes')

        choice(name: 'CLD_PROVIDER', choices: ['gcp'], description: 'Select the Cloud Provider')

        string(name: 'STREAMFLAKE_VERSION', defaultValue: '', description: 'Select the Cloud Provider')

        booleanParam(name: 'RELEASE_ARTIFACT', defaultValue: false, description: 'Note-Need to be updated post s3 bucket creation') //update description once separate bucket has been created for streamflake

        choice(name: 'BASE_DOWNLD_LOC', choices: ['s3://streamflake-nightly','s3://archives.streamsets.com/streamflake'], description: 'Select base download bucket') //update choices once separate bucket has been created for streamflake
        
        string(name: 'BRANCH_NAME', defaultValue: '', description: 'Enter branch name')

        string(name: 'SPARK_VERSION', defaultValue: 'spark-2.4.8', description: 'Enter spark version')

        string(name: 'PACKER_VERSION', defaultValue: '1.8.3', description: 'Enter packer version')

        string(name: 'COMMIT_ID', defaultValue: '', description: 'Please Add the Git Commit ID of the product here')
        
        string(name: 'SPARK_ENGINE_VER', defaultValue: '2.12', description: 'Enter spark engine version')

        string(name: 'PROJECT_ID', defaultValue: 'gcp-probe', description: 'Enter GCP project ID where we need to create image')
    }

    stages{
        stage("RELOAD JOB"){
            steps{
                script {
                    echo "Jenkins job reloaded with latest jenkinsfile changes"
                    currentBuild.getRawBuild().getExecutor().interrupt(Result.SUCCESS)
                    sleep(1)
                }
            }
            when {
                expression { RELOAD_JOB == "true" }
            }
        }
        stage("Installing gcloud cli") {
            steps {
                script {
                    sh '''
                        #!/bin/bash -ex
                        curl -sS https://sdk.cloud.google.com > /tmp/install.sh
                        $HOME/google-cloud-sdk/bin/gcloud version >/dev/null 2>&1 || bash /tmp/install.sh --disable-prompts
                    '''
                }
            }
        }
        stage("Installing packer") {
            steps {
                script {
                    sh '''
                        #!/bin/bash -ex
                        wget -q https://releases.hashicorp.com/packer/${PACKER_VERSION}/packer_${PACKER_VERSION}_linux_amd64.zip
                        unzip -o packer_${PACKER_VERSION}_linux_amd64.zip
                        rm -rf packer_${PACKER_VERSION}_linux_amd64.zip
                    '''
                }
            }
        }
        stage("Creating GCP image name") {
            steps {
                script {
                    DASH_STREAMFLAKE_VERSION = STREAMFLAKE_VERSION.replaceAll('\\.','-').toLowerCase()
                    TIMESTAMP = sh (script: 'date +"%m%d%y-%H%M%S"', returnStdout: true)
                    GCP_STREAMFLAKE_IMAGE_NAME = "streamflake-csp-v2-12-r${DASH_STREAMFLAKE_VERSION}-${TIMESTAMP}"
                    echo "GCP_STREAMFLAKE_IMAGE_NAME: ${GCP_STREAMFLAKE_IMAGE_NAME}"
                }
            }
        }
        stage("Downloading tarfile"){
            steps{
                script {
                    if (RELEASE_ARTIFACT == "true") {
                        DOWNLOAD_S3_URI = "${BASE_DOWNLD_LOC}/${BRANCH_NAME}/${SPARK_ENGINE_VER}/tarball/${STREAMFLAKE_TARFILE}"
                    } else {
                        DOWNLOAD_S3_URI = "${BASE_DOWNLD_LOC}/${SPARK_ENGINE_VER}/tarball/${STREAMFLAKE_TARFILE}"
                    }

                    echo "DOWNLOAD_S3_URI: ${DOWNLOAD_S3_URI}"
                }
            }
        }
        stage("Echo all vars") {
            steps {
                script {
                    sh(script: 'bash test.sh',returnStdout: true)
                }
            }
        }
    }
}
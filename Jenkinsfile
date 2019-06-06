pipeline {
    agent {
        node {
            label 'master'
            
        }
    }
    environment {
      PATH="/extdrive3/software/anaconda2/bin:$PATH"
    }
    stages {
        stage('Clone Focalpoint from github'){
          steps{
                echo 'clean up workspace'
                cleanWs()
                echo 'test1'
                checkout([$class: 'GitSCM', branches: [[name: '*/${branch_name}']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'e8b16435-53fc-4510-8387-441c708a7c86', url: 'git@github.ibm.com:Augusta/FocalPointAttribution.git']]])
                }
        }
        stage('Create New Virtual Environment'){
          steps{
                echo 'Create New Virtual Environment'
                sh '''conda create --clone jenkins_base -n ${BUILD_TAG}
	                    which pip
	                    source activate ${BUILD_TAG}
	                    which pip
	                    pip install -r requirements.txt .[keyring]
	                    pip install xlsxwriter
	                    pip install xlrd
                	'''
                }
        }
        stage('Test environment') {
            steps {
                sh '''source activate ${BUILD_TAG}
                      pip list
                      which pip
                      which python
                      ls -lah ~/.local
                    '''
            }
        }
        stage('Run API1A'){
            steps{
                echo 'API1A'
                sh '''source activate ${BUILD_TAG}
                      python src/focalpoint/nonspark/data_curation/pull_claim_enroll_monthly.py ${params_dir}${data_source}/MarketScan/data_curation.params  --logging_level=INFO
                      python src/focalpoint/nonspark/data_curation/create_claim_enroll_blocks.py ${params_dir}${data_source}/MarketScan/data_curation.params ${params_dir}${data_source}/MarketScan/experiment.params --logging_level=INFO
                    '''
            }
        }
        stage('Run API1B'){
            steps{
                echo 'API1B'
                sh '''source activate ${BUILD_TAG}
                    python src/focalpoint/nonspark/data_curation/create_meg_opeg_blocks.py ${params_dir}${data_source}/MarketScan/data_curation.params ${params_dir}${data_source}/MarketScan/experiment.params --logging_level=ERROR
                    python src/focalpoint/nonspark/data_curation/join_meg_opeg_claims.py ${params_dir}${data_source}/MarketScan/data_curation.params ${params_dir}${data_source}/MarketScan/experiment.params --logging_level=ERROR
                '''
            }
        }
        stage('Run API2A'){
            steps{
                echo 'API2A'
                sh '''source activate ${BUILD_TAG}
                    mkdir -p output/dfcost
                    python src/focalpoint/nonspark/analytics/api_claim_merge_aggregate.py expts/ ${params_dir}${data_source}/MarketScan/ src/focalpoint/common/mapping/ output/dfcost/ short ${viewpoints} 5 --save_claims &
                    python src/focalpoint/nonspark/analytics/api_claim_merge_aggregate.py expts/ ${params_dir}${data_source}/MarketScan/ src/focalpoint/common/mapping/ output/dfcost/ long ${viewpoints} 5 --save_claims
                '''
            }            
        }
        stage('Run API2B'){
            steps{
                echo 'API2B'
                sh '''source activate ${BUILD_TAG}
                    mkdir -p output/dfdetect
                    python src/focalpoint/nonspark/analytics/api_cusum_nd.py ${params_dir}${data_source}/MarketScan/ output/dfcost/ output/dfdetect/ short ${viewpoints} &
                    python src/focalpoint/nonspark/analytics/api_cusum_nd.py ${params_dir}${data_source}/MarketScan/ output/dfcost/ output/dfdetect/ long ${viewpoints}
                '''
            }
        }
        stage('Run API2C'){
            steps{
                echo 'API2C'
                sh '''source activate ${BUILD_TAG}
                    mkdir -p output/dffinal
                    python src/focalpoint/nonspark/analytics/api_2c_move_other_direction.py ${params_dir}${data_source}/MarketScan/ src/focalpoint/common/mapping/ output/dfcost/ output/dfdetect/ output/dffinal/ short ${viewpoints} &
                    python src/focalpoint/nonspark/analytics/api_2c_move_other_direction.py ${params_dir}${data_source}/MarketScan/ src/focalpoint/common/mapping/ output/dfcost/ output/dfdetect/ output/dffinal/ long ${viewpoints}
                '''
            }
        }
        stage('Run API3'){
            steps{
                echo 'API3'
                sh '''source activate ${BUILD_TAG}
                        pip list
                        pip install xlsxwriter
                        pip install xlrd
                        pip list
                        mkdir report
                        python src/focalpoint/nonspark/analytics/api_post_processing.py "${params_dir}${data_source}/MarketScan/" "output/dffinal/" "report/" "expts/" --pull
                '''
            }
        }
        stage('ZIP artifacts') {
            steps {
                sh 'zip -r artifacts.zip data/ expts/ output/ report/'
            }
        }
    }
    
}



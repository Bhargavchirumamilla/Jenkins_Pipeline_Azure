import com.sap.ms.*
def log = new com.sap.ms.L()
def productBuildName
def codeCoverageReportLoc = ""
def deployFailed=false
def thisQualStartTime = new Date()

import com.sap.ms.*

if (env.BRANCH_NAME != 'manage_quals') { return }

node("swarm") {

    // Product Speciifc Parameters 
    def en = new com.sap.ms.Env()
    def productNames     = new StringParameterDefinition('Product_Name', 'minerva', 'Please Enter Product Name - Exmaple: minerva')
    def serviceNames     = new ChoiceParameterDefinition('Service_Name', ['gcpman02','gcpman03'] as String[], 'Please Enter Service Name - Example: gcpman02')
    def productBranch    = new StringParameterDefinition('Product_Branch', 'develop', 'Please Enter Product Branch - Exmaple: trunk')
    def pipelineName     = new StringParameterDefinition('Pipeline_Name', 'Ariba-Sourcing', 'Please Product Pipeline Path')
    def buildNumber      = new StringParameterDefinition( 'BUILD_NUMBER_EX', 'LATEST', 'Please enter the build number to deploy')

    // Deploy Service Specific Parameters
    def deployService    = new BooleanParameterDefinition('DEPLOY_SERVICES', true, '')
    def productPath      = new StringParameterDefinition( 'PRODUCT_PATH', 'ariba-cobalt-minerva', 'Example: ariba-sampleapp')
    def clusterIP        = new ChoiceParameterDefinition('CLUSTER_IP', getAllElbs{} , 'Enter cluster IP address. This take precedence over ELB parameter')
    def customMode       = new ChoiceParameterDefinition('CUSTOM_MODE', ['itg-gcpman02'] as String[] , 'Enter Cluster Mode')
    def queryName        = new ChoiceParameterDefinition('QUERY', ['sourcingms-gcpman02'] as String[], 'Query name is optional, value from the build config will be used when not provided.')
    def customRegistry   = new ChoiceParameterDefinition('CUSTOM_REGISTRY', getAllRegistries{} , 'Enter mode. This take precedence over MODE parameter')
    def custArtifactRepo = new ChoiceParameterDefinition('CUSTOM_ARTIFACTORY_URL', en.getAllArtifactory(), 'Enter artifactory url. This take precedence over ARTIFACTORY_URL parameter')

    def clusterKind      = new ChoiceParameterDefinition('CLUSTER_TYPE', ['hashi','kubernetes'] as String[], 'Cluster type k8s/hashi')
    def landscapes       = new ChoiceParameterDefinition('LANDSCAPE', ['','dev','qa','itg','perf','e2e'] as String[], 'Target namespace within the cluster')
    def custLandscape    = new StringParameterDefinition('CUSTOM_LANDSCAPE', '', 'Enter namespace. This take precedence over LANDSCAPE parameter')
    def familyNamespace  = new StringParameterDefinition('FAMILY_NAMESPACE', '', 'Enter family namespace.')

    def modePrecedence   = new BooleanParameterDefinition('MODE_PRECEDENCE', false, '')
    def skipRollback     = new BooleanParameterDefinition('SKIP_ROLLBACK', false, '')
    def markAsDefault    = new BooleanParameterDefinition('MARK_AS_DEFAULT', true, '')
    def internal         = new BooleanParameterDefinition('INTERNAL', false, 'select if the service is required to accessed through internal front door')
    def maxUploadSize    = new BooleanParameterDefinition('MaxUploadSize', false, 'select to update max upload size')
    def uploadlimit      = new StringParameterDefinition('Uploadlimit', '1024m', 'E.g. 1024m')

    // Katalon Parameters
    def katalonQual      = new BooleanParameterDefinition('KATALON_QUAL', true, 'Run Katalon Qual')
    def katalonVersion   = new ChoiceParameterDefinition('Katalon_Version', ['8.0.1','default'] as String[], 'Select Katalon Version')
    def qualType         = new ChoiceParameterDefinition('Qual_Type', ['IC','CQ','QI','collections','suites'] as String[], 'Select Qual Type')
    def components       = new StringParameterDefinition('Components', 'MinervaKatalonTest' , 'Katalon Project Name')
    def collectionsPath  = new StringParameterDefinition('Collections_or_Suites', 'Test Suites/Collections/IC/API_Collection.ts' , 'Collection or Suite Path')
    def testRepoName     = new StringParameterDefinition('Test_Repo_Branch', 'develop' , 'Katalon Test Repo Branch')
    def executerEngine   = new StringParameterDefinition('Executer_Engine_Repo_Branch', 'master' , 'Katalon Executer Engine Repo Branch')
    def customization    = new StringParameterDefinition('Customization_Repo_Branch', 'master' , 'Katalon Customization Repo Branch')
    def propertiesName   = new StringParameterDefinition('Properties_Repo_Branch', 'master' , 'Katalon Properties Repo Branch')
    def qtestUpload      = new BooleanParameterDefinition('Upload_Result_To_QTest', true, 'Upload Katalon Result to QTest')

    // Code Coverage Parameters
    def codeCoverage     = new BooleanParameterDefinition('CODE_COVERAGE', true, 'Collect Code Coverage for Integration Test')
    def junitCodeCoverage= new BooleanParameterDefinition('Include_Product_Junit_Coverage', true, 'Collect Code Coverage for Integration Test')
    
    properties([])
    properties([ 
        [$class: 'ParametersDefinitionProperty', parameterDefinitions: [ 
            productNames, serviceNames, productBranch, pipelineName, buildNumber,
            deployService, productPath, clusterIP, customMode, queryName, customRegistry, custArtifactRepo, 
            clusterKind, landscapes, custLandscape, familyNamespace, 
            modePrecedence, skipRollback, markAsDefault, internal, maxUploadSize, uploadlimit,
            katalonQual, katalonVersion, qualType, components, collectionsPath, testRepoName, executerEngine, customization, propertiesName, qtestUpload, 
            codeCoverage, junitCodeCoverage
        ]], 
        pipelineTriggers([ 
            parameterizedCron('''
            0 3 * * * %Service_Name=gcpman02;CLUSTER_IP=itg-ncv.cobalt.ariba.com;CUSTOM_MODE=itg-gcpman02;QUERY=sourcingms-gcpman02
            ''')
        ]) 
    ])

    log.info("####### Product Information ######################")
    log.info("Product_Name          : ${params.Product_Name}")
    log.info("Service_Name          : ${params.Service_Name}")
    log.info("Product_Branch        : ${params.Product_Branch}")
    log.info("Pipeline_Name         : ${params.Pipeline_Name}")
    log.info("DEPLOY_SERVICES       : ${params.DEPLOY_SERVICES}")
    log.info("KATALON_QUAL          : ${params.KATALON_QUAL}")
    log.info("CODE_COVERAGE         : ${params.CODE_COVERAGE}")
    log.info("##################################################")
}

def thisProductName = "${params.Product_Name}"
def thisQualType = "${params.Qual_Type}"
def thisServiceName = "${params.Service_Name}"
def thisComponents = "${params.Components}"
def thisCollections_or_Suites = "${params.Collections_or_Suites}"
def thisConfig = "${params.Executer_Engine_Repo_Branch}"
def thisCustomization = "${params.Customization_Repo_Branch}"
def thisProperties = "${params.Properties_Repo_Branch}"
def thisTestRepo = "${params.Test_Repo_Branch}"
def thisKatalonVersion = "${params.Katalon_Version}"
def thisClusterIP = "${params.CLUSTER_IP}"
def thisuploadToQTest = params.Upload_Result_To_QTest

def thisRegistry = "${params.CUSTOM_REGISTRY}"
def thisProductPath = "${params.PRODUCT_PATH}"
def thisQuery = "${params.QUERY}"
def thisIncludeJunitCoverage = params.Include_Product_Junit_Coverage
def thisProductPipelinePath = "${env.JENKINS_URL}/job/${params.Pipeline_Name}/job/${params.Product_Name}/job/${params.Product_Branch}"

if(isSCMTrigger{}) { return }

pipeline {

    agent { label 'perl' }

    stages {

        stage("Get Build") { 
            steps { 
            script {
                if(!params.Product_Name){
                    currentBuild.result = "FAILURE"
                    error('Stopping early, Product_Name is Empty')
                }
                log.info("")
                log.info("########################################################################")
                log.info("STAGE: Get Build Name")
                log.info("########################################################################")

                catchError {
                    if ( params.BUILD_NUMBER_EX == 'LATEST' || params.BUILD_NUMBER_EX == '' ) {
                        log.info("Get Latest Stable Build Name")
                        def buildState = 'lastStableBuild';
                        log.info("Product_Name          : ${params.Product_Name}")
                        log.info("Product_Branch        : ${params.Product_Branch}")
                        log.info("Pipeline_Name         : ${params.Pipeline_Name}")
                        def displayNameUrl = "${thisProductPipelinePath}/${buildState}/api/json?tree=displayName"
                        log.info("Getting ${buildState} using: ${displayNameUrl}")
                        def response = Curl.GetJsonData(displayNameUrl)
                        productBuildName = response.displayName
                        log.info("Build for " + params.Product_Branch + ' is : ' + productBuildName);
                    } else {
                        productBuildName = "${params.BUILD_NUMBER_EX}"
                        log.info("Build Name is : " + productBuildName);
                    }
                    currentBuild.displayName = "${params.Product_Name}-${params.Service_Name}-${params.Product_Branch}-${productBuildName}"
                }
            } 
            } 
        }

        stage("Deploy Build") {
            when { expression { params.DEPLOY_SERVICES ==~ /(true)/ } }
            steps {
            script {
                log.info("")
                log.info("########################################################################")
                log.info("STAGE: Deploy Build ${productBuildName} in ${params.CLUSTER_IP} in Progress")
                log.info("########################################################################")
        
                log.info("DEPLOY_SERVICES       : ${params.DEPLOY_SERVICES}")
                log.info("PRODUCT_PATH          : ${params.PRODUCT_PATH}")
                log.info("BUILD_NUMBER_EX       : ${productBuildName}")
                log.info("CLUSTER_IP            : ${params.CLUSTER_IP}")
                log.info("CUSTOM_REGISTRY       : ${params.CUSTOM_REGISTRY}")
                log.info("CUSTOM_MODE           : ${params.CUSTOM_MODE}")
                log.info("CUSTOM_ARTIFACTORY_URL: ${params.CUSTOM_ARTIFACTORY_URL}")
                log.info("MODE_PRECEDENCE       : ${params.MODE_PRECEDENCE}")
                log.info("SKIP_ROLLBAC  K       : ${params.SKIP_ROLLBACK}")
                log.info("MARK_AS_DEFAULT       : ${params.MARK_AS_DEFAULT}")
                log.info("QUERY                 : ${params.QUERY}")
                log.info("INTERNAL              : ${params.INTERNAL}")
                log.info("MaxUploadSize         : ${params.MaxUploadSize}")
                log.info("Uploadlimit           : ${params.Uploadlimit}")
        
                def jobBuild=build job: "Ariba-pipelines/tools/AppDeploy",  
                parameters: [ 
                    string(name: 'PRODUCT_PATH', value: params.PRODUCT_PATH),
                    string(name: 'BUILD_NUMBER_EX', value: productBuildName),
                    string(name:'CLUSTER_IP',value: params.CLUSTER_IP), 
                    string(name:'CUSTOM_MODE',value: params.CUSTOM_MODE), 
                    string(name:'QUERY',value: params.QUERY),
                    string(name:'CUSTOM_REGISTRY',value: params.CUSTOM_REGISTRY), 
                    string(name:'CUSTOM_ARTIFACTORY_URL',value: params.CUSTOM_ARTIFACTORY_URL), 
                    string(name:'CLUSTER_TYPE',value: params.CLUSTER_TYPE), 
                    string(name:'LANDSCAPE',value: "${params.LANDSCAPE}"), 
                    string(name:'CUSTOM_LANDSCAPE',value: params.CUSTOM_LANDSCAPE), 
                    string(name:'FAMILY_NAMESPACE',value: params.FAMILY_NAMESPACE), 
                    booleanParam(name:'MODE_PRECEDENCE',value: params.MODE_PRECEDENCE),
                    booleanParam(name:'SKIP_ROLLBACK',value: params.SKIP_ROLLBACK),
                    booleanParam(name:'MARK_AS_DEFAULT',value: params.MARK_AS_DEFAULT),
                    booleanParam(name:'INTERNAL',value: params.INTERNAL),
                    booleanParam(name:'MaxUploadSize',value: params.MaxUploadSize),
                    string(name:'Uploadlimit',value: params.Uploadlimit), 
                ],propagate: false;
     
                echo "URL:"+ jobBuild.getAbsoluteUrl();
                def jobResult = jobBuild.getResult()
                if (jobResult != 'SUCCESS') {
                    echo "Build returned result: ${jobResult}"
                    error("Deploy Failed ... Exiting")
                    deployFailed=true
                } else {
                   echo "Build returned result: ${jobResult}"
                   log.info("Build ${productBuildName} Deploy in ${params.CLUSTER_IP} Completed Successfully!!")
                }
            }
            }
        }   

        stage("Katalon Qual") {
            when { expression { params.KATALON_QUAL ==~ /(true)/ && deployFailed ==~ /(false)/ }}
            steps {
            script {
                catchError {
                    log.info("")
                    log.info("########################################################################")
                    log.info("STAGE: Katalon Qual")
                    log.info("########################################################################")
    
                    runKatalonTests {
                        product = "${thisProductName}"
                        qualType = "${thisQualType}"
                        buildName = "${productBuildName}"
                        serviceName = "${thisServiceName}"
                        components = "${thisComponents}"
                        collectionsOrSuites = "${thisCollections_or_Suites}"
                        configurationBranch = "${thisConfig}"
                        customizationBranch = "${thisCustomization}"
                        propertiesBranch = "${thisProperties}"
                        componentsBranch = "${thisTestRepo}"
                        reportConsolidation=false
                        setupKatalon=true 
                        reportOnly=false
                        katalonVersion = "${thisKatalonVersion}"
                        qualStartTime = "$thisQualStartTime"
                        uploadKatalonToQtest = thisuploadToQTest
                        optionalArgs = "Cluster_Name::${thisClusterIP}"
                    }
                }
            }
            }
        }

        stage("Code Coverage") {
            when { expression { params.CODE_COVERAGE ==~ /(true)/ } }
            steps {
            script {
                cleanWs()
                catchError {
                    log.info("")
                    log.info("########################################################################")
                    log.info("STAGE: Code Coverage : Collect Application Jars, Coverage Dump and Publish Report")
                    log.info("########################################################################")

                    runCodeCoverage {
                        collectAppClasses = true
                        collectCoverage = true
                        includeJunitCoverage = thisIncludeJunitCoverage
                        clusterName = "${thisClusterIP}"
                        productPathName = "${thisProductPath}"
                        registryName = "${thisRegistry}"
                        queryName = "${thisQuery}"
                        buildName = "${productBuildName}"
                        productPipelinePath = "${thisProductPipelinePath}"
                        containerNames = ["minerva-app"]
                    }
                    codeCoverageReportLoc="${env.BUILD_URL}/jacoco/"
                 }
            }
            }
        }
    } 
    post {
        success {
        script {
            if ( params.KATALON_QUAL == true && deployFailed == false ) {   
                log.info("")
                log.info("########################################################################")
                log.info("STAGE: Generate Consolidated Report and Send Mail")
                log.info("########################################################################")
      
                def thisQualEndTime = new Date()
                runKatalonTests {
                    product = "${thisProductName}"
                    qualType = "${thisQualType}"
                    buildName = "${productBuildName}"
                    serviceName = "${thisServiceName}"
                    components = "${thisComponents}"
                    collectionsOrSuites = "${thisCollections_or_Suites}"
                    configurationBranch = "${thisConfig}"
                    customizationBranch = "${thisCustomization}"
                    propertiesBranch = "${thisProperties}"
                    componentsBranch = "${thisTestRepo}"
                    setupKatalon=false
                    reportConsolidation=true
                    reportOnly=true
                    katalonVersion = "${thisKatalonVersion}"
                    qualStartTime = "$thisQualStartTime"
                    qualEndTime = "$thisQualEndTime"
                    codeCoverageReportLink = "${codeCoverageReportLoc}"
                    uploadKatalonToQtest = thisuploadToQTest
                    optionalArgs = "Cluster_Name::${thisClusterIP}"
                }
            }
            log.info("Succeeded!")
        }
        }
    }
}



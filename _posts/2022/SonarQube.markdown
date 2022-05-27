# Add a project in Sonarqube Manually

#### Step 1: Create the new project in Sonarqube

Click "*new project*" and choose "*manually*".

#### Step 2: Setup the project info

Add "*Project display name*" and "*Project key*"

> **Project display name**: Name of the project is shown on a Sonarqube. *Up to 255 characters. Some scanners might override the value you provide.*
>
> **Project key**: Unique key to connect the source code with the Sonarqube project. *The project key is a unique identifier for your project. It may contain up to 400 characters. Allowed characters are alphanumeric, '-' (dash), '_' (underscore), '.' (period) and ':' (colon), with at least one non-digit.*

#### Step 3: Add or Collect the Sonar HOST_URL and LOGIN_Key in AWS Parameter Store

Collect the Sonar HOST_URL and LOGIN_Key in [AWS Systems Manager - Parameter Store (amazon.com)](https://ap-southeast-1.console.aws.amazon.com/systems-manager/parameters/?region=ap-southeast-1&tab=Table#list_parameter_filters=Name:Contains:sonarqube)

*Note: These info will use to connect the Sonarqube projects from the source codes.*

#### Step 4: Create the folder in S3 bucket

Create the new folder in S3 bucket to store the .xml testing files.
Eg: Go to the [yomafleet-github-artifacts - S3 bucket (amazon.com)](https://s3.console.aws.amazon.com/s3/buckets/yomafleet-github-artifacts?region=ap-southeast-1&tab=objects) and create the carshare-webhook/ folder and choose Server-side encryption is disabled.

> **Folder**: Folder names can't contain "/" (It will generate the S3 url like [s3://yomafleet-github-artifacts/carshare-webhook/]()
>
> **Server-side encryption**: The following settings apply only to the new folder object and not to the objects contained within it. (Attribute: Disable or Enable)

#### Step 5: Create the configuration files for Sonarqube in the source code

Create the sonar-project.properties and add the following codes.

```terraform
#must be unique in a given SonarQube instance_
sonar.projectKey=carshare-webhook-staging
#--- optional properties ---_
#defaults to project key_
sonar.projectName=Yoma Fleet Car Share WebHook Staging
#defaults to 'not provided'_
sonar.projectVersion=0.1.0
#Path is relative to the sonar-project.properties file. Defaults to ._
sonar.sources=app
sonar.tests=tests
#Encoding of the source code. Default is default system encoding_
sonar.sourceEncoding=UTF-8
#PHP Exclustions_
sonar.php.exclusions=**/vendor/**
sonar.exclusions=bootstrap/cache/*, public/vendor/**, resources/lang/**
#PHP Unit Test Properties_
#sonar.php.coverage.reportPaths=reports/coverage.xml_
#sonar.php.tests.reportPath=reports/tests-report.xml_
##Flex Specific Properties_
#sonar.dependencyCheck.jsonReportPath=reports/dependency-check-report.json_
#sonar.dependencyCheck.xmlReportPath=reports/dependency-check-report.xml_
#sonar.dependencyCheck.htmlReportPath=reports/dependency-check-report.html_
```

Change the following variables based on the project:

> **sonar.projectkey**: Add the project key on the above info when we created the project in Sonarqube.

Create the buildspec_sast.ymal to build the source code in AWS CodeBuild  as the following:

    version:  0.2
    
    env:
    parameter-store:
    SONAR_HOST_URL:  /CodeBuild/YomaFleet/SonarQube/SONAR_HOST_URL
    SONAR_LOGIN:  /CodeBuild/YomaFleet/SonarQube/SONAR_LOGIN
    
    phases:
    pre_build:
    on-failure:  CONTINUE
    commands:
    -  echo "Downdload PHPUnit Test Report & Coverage"
    -  aws --version
    -  mkdir -p $PWD/reports
    -  cd reports/
    -  aws s3 cp s3://yomafleet-github-artifacts/carshare-webhook/coverage.xml .
    -  aws s3 cp s3://yomafleet-github-artifacts/carshare-webhook/tests-report.xml . 
    finally: 
    -  echo "Insert/Replace Paths with SonarQube Started" 
    -  sed -i 's/home\/runner\/work\/carshare-webhook\/carshare-webhook/usr\/src/g' coverage.xml 
    -  sed -i 's/home\/runner\/work\/carshare-webhook\/carshare-webhook/usr\/src/g' tests-report.xml  
    -  echo "Insert/Replace Paths with SonarQube Completed"  
    build: 
    commands:
    -  cd ..
    -  echo "SonarQube Scan"
    -  docker run --rm -e SONAR_HOST_URL=$SONAR_HOST_URL -e SONAR_LOGIN=$SONAR_LOGIN -v "$PWD:/usr/src" sonarsource/sonar-scanner-cli
    finally:
    -  echo "SonarQube Scan Completed"
    post_build:
    commands:
    -  echo Entered the post_build phase...
    -  echo Build completed on `date`
    artifacts:
    files:
    -  '**/*'
    name:  BuildArtifacts

`ddddddd`
`dddddd`

Written with [StackEdit](https://stackedit.io/).
Created by Hein Htet Nyunt Win

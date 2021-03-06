
Sample Azure Yaml consists of 
Build -> Test -> Deploy Jobs (test123)

```
trigger:
- master

pool:
  vmImage: 'ubuntu-latest'

name: $(Date:yyyyMMdd)$(Rev:.r)
jobs:
- job: 'Build'
  steps:
  - task: ServiceNow Job Notification@0
    inputs:
      connectedServiceName: 'TarunDevOpsMadrid'
      Phase: 'STARTED'
  - task: Maven@3
    name: 'build'
    displayName: 'Build'
    inputs:
      mavenPomFile: 'pom.xml'
      mavenOptions: '-Xmx3072m'
      javaHomeOption: 'JDKVersion'
      jdkVersionOption: '1.8'
      jdkArchitectureOption: 'x64'
      publishJUnitResults: false
      testResultsFiles: '**/TEST-*.xml'
      goals: 'clean install'
  - task: ServiceNow Job Notification@0
    inputs:
      connectedServiceName: 'TarunDevOpsMadrid'
      Phase: 'COMPLETED'
- job: 'Test'
  dependsOn: 'Build'
  steps:
  - task: ServiceNow Job Notification@0
    inputs:
      connectedServiceName: 'TarunDevOpsMadrid'
      upstream: 'Build'
      Phase: 'STARTED'
  - task: Maven@3
    name: 'test'
    displayName: 'Test'
    inputs:
      mavenPomFile: 'pom.xml'
      mavenOptions: '-Xmx3072m'
      javaHomeOption: 'JDKVersion'
      jdkVersionOption: '1.8'
      jdkArchitectureOption: 'x64'
      publishJUnitResults: false
      testResultsFiles: '**/TEST-*.xml'
      goals: 'test'
  - task: ServiceNow Job Notification@0
    inputs:
      connectedServiceName: 'TarunDevOpsMadrid'
      upstream: 'Build'
      Phase: 'COMPLETED'
- job: 'Deploy'
  dependsOn: 'Test'
  steps:
  - task: ServiceNow Job Notification@0
    inputs:
      connectedServiceName: 'TarunDevOpsMadrid'
      upstream: 'Test'
      Phase: 'STARTED'
  - task: Maven@3
    name: 'deploy'
    displayName: 'Deploy'
    inputs:
      mavenPomFile: 'pom.xml'
      mavenOptions: '-Xmx3072m'
      javaHomeOption: 'JDKVersion'
      jdkVersionOption: '1.8'
      jdkArchitectureOption: 'x64'
      publishJUnitResults: false
      testResultsFiles: '**/TEST-*.xml'
      goals: 'package'
  - task: ServiceNow Job Notification@0
    inputs:
      connectedServiceName: 'TarunDevOpsMadrid'
      upstream: 'Test'
      Phase: 'COMPLETED'
```

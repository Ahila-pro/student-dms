trigger:
- main

pool:
  vmImage: ubuntu-latest

variables:
  phpVersion: 8.2
  packagePath: '$(Build.ArtifactStagingDirectory)/laravel-app.zip'

steps:
# Install PHP and dependencies
- task: UsePhpVersion@0
  inputs:
    version: '$(phpVersion)'

- script: |
    php -v
    composer install --no-interaction --prefer-dist --optimize-autoloader
    cp .env.example .env
    php artisan key:generate
  displayName: 'Install PHP dependencies & setup Laravel'

# Archive the application
- task: ArchiveFiles@2
  inputs:
    rootFolderOrFile: '$(System.DefaultWorkingDirectory)'
    includeRootFolder: false
    archiveType: 'zip'
    archiveFile: '$(packagePath)'
    replaceExistingArchive: true
  displayName: 'Archive Laravel app'

# Publish artifact
- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'Container'

# Deploy to Azure App Service
- task: AzureRmWebAppDeployment@5
  displayName: 'Deploy to Azure App Service'
  inputs:
    ConnectionType: 'AzureRM'
    azureSubscription: 'Azure subscription 1(1)(8ca97cad-9ab1-42a2-9d9e-bdbf8876c147)'
    appType: 'webAppLinux'
    WebAppName: 'studdmsstatic'
    deployToSlotOrASE: true
    ResourceGroupName: 'rgp-devops'
    SlotName: 'production'
    packageForLinux: '$(packagePath)'
    RuntimeStack: 'PHP|8.2'
    StartupCommand: ''
    DeploymentTypeLinux: 'oneDeploy'

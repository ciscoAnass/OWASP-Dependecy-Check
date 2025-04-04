# OWASP Dependecy Check

# Dependecy Check for .NET Projects

## Define the pipeline trigger

```yml
trigger:
- main
```

- *Explanation:* The pipeline will automatically run whenever changes are pushed to the main branch.

## Define the Virtual Machine Image

```yml
pool:
  vmImage: 'ubuntu-latest'
```

- *Explanation:* The pipeline will run on a virtual machine with Ubuntu-latest.

## Run Simple Script

```yml
steps:
- script: echo Hello, world!
  displayName: 'Run a one-line script'
```

- *Explanation:* Print Hello World!!

## Build .NET project phase

```yml
- task: DotNetCoreCLI@2
  displayName: 'Build the project'
  inputs:
    command: 'build'
    projects: '**/*.sln'
    arguments: '--configuration Release'
```

- We are going to scan C# Codes so we are using NetCore , if you wanna scan any other code (Python, Ruby, Java, JS,....) check this Page : [Build Phase](/files/1-build.md)

## Publish the project artifact

```yml
- task: DotNetCoreCLI@2
  displayName: 'Create artifact'
  inputs:
    command: 'publish'
    publishWebProjects: true
    arguments: '--configuration Release --output $(Build.ArtifactStagingDirectory)'
    zipAfterPublish: true
```

- We are scanning **C#** files so we are using **DotNetCoreCLI** , if you wanna publish the project artifact in other languages check this [Link](/files/2-publish.md)

## Create a directory for reports

```yml
- task: Bash@3
  displayName: 'Create Report Directory'
  inputs:
    targetType: 'inline'
    script: 'mkdir -p $(Build.ArtifactStagingDirectory)/dependency-check-report'
```

- Uses a Bash script to create a directory for dependency check reports.

## Run OWASP Dependency Check

```yml
- task: dependency-check-build-task@6
  displayName: 'Run OWASP Dependency Check'
  inputs:
    projectName: 'Clean.DDD.Architecture'
    scanPath: '$(Build.SourcesDirectory)'
    format: 'HTML'
    reportPath: '$(Build.ArtifactStagingDirectory)/dependency-check-report'
    nvdApiKey: 'Your API'
    outputDirectory: '$(Build.ArtifactStagingDirectory)/dependency-check-report'

```

- Explanation:
- ---> Runs OWASP Dependency Check to scan for vulnerabilities in the dependencies.
- ---> Generates an HTML report.
- ---> Saves the report in $(Build.ArtifactStagingDirectory)/dependency-check-report.
- ---> Uses an API key for accessing the National Vulnerability Database (NVD).

## Publish the HTML report

```yml
- task: PublishHtmlReport@1
  displayName: 'Publish Dependency Check HTML Report'
  inputs:
    reportDir: '/home/vsts/work/1/TestResults/dependency-check'
    tabName: 'Dependency Check Report'
    reportHtml: 'dependency-check-report.html'
```

- Publishes the generated dependency check report to the pipeline UI.

## Display the HTML report content in logs

```yml
- task: CmdLine@2
  displayName: 'Show HTML Report in Pipeline Logs'
  inputs:
    script: |
      echo "==================== HTML REPORT CONTENT ===================="
      cat /home/vsts/work/1/TestResults/dependency-check/dependency-check-report.html
      zip -r /home/vsts/work/1/TestResults/dependency-check/dependency-check-report.zip /home/vsts/work/1/TestResults/dependency-check/dependency-check-report.html
```

- Displays the HTML report content in the pipeline logs.
- Compresses the HTML report into a ZIP file.

## Publish the ZIP file containing the report

```yml
- task: PublishBuildArtifacts@1
  displayName: 'Publish ZIP file of HTML Report'
  inputs:
    PathtoPublish: '/home/vsts/work/1/TestResults/dependency-check/dependency-check-report.zip'
    ArtifactName: 'dependency-check-report.zip'
```

## Publish the full dependency check report

```yml
- task: PublishBuildArtifacts@1
  displayName: 'Publish Dependency Check Report'
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)/dependency-check-report'
    ArtifactName: 'dependency-check-report'
```

To get the full-code check this [Link](/files/cplus.yml)

---

# Azure Pipeline : Pip-audit for Python

```yml
trigger:
- develop

pool:
  vmImage: ubuntu-latest

steps:
- script: echo Hello, world!
  displayName: 'Run a one-line script'

# Instalar los dependecies de Python
- task: PythonScript@0
  displayName: 'COnfigurar el entorno de Python | import subprocess y sys | la version | isntalar bandit y pip-audit | '
  inputs:
    scriptSource: 'inline'
    script: |
      import subprocess
      import sys
      print(f"Python version: {sys.version}")

      subprocess.run(["python", "--version"])
      subprocess.run(["python3", "--version"])

      subprocess.run(["pip3", "install", "bandit"])
      subprocess.run(["pip", "install", "pip-audit"])
      subprocess.run(["pip", "install", "-r", "requirements.txt"])

# Ejecutar bandit por escaneo de seguridad (dependecy check)
- task: PythonScript@0
  displayName: 'Bandit por dependecy check'
  inputs:
    scriptSource: 'inline'
    script: |
      import subprocess
      subprocess.run(["bandit", "-r", "."])

# Ejecutar pip-audit y mostrar el resultado 
- task: Bash@3
  displayName: 'Ejecutar pip-audit y mostrar resultados'
  inputs:
    targetType: 'inline'
    script: |
      pip-audit
```

---

## Azure Pipeline : Pip-audit for Python with Zip Option

```yml
trigger:
- develop

pool:
  vmImage: ubuntu-latest

steps:
- script: echo Hello, world!
  displayName: 'Run a one-line script'

# Crear entorno virtual
- task: Bash@3
  displayName: 'Crear un entorno virtual se llama anass'
  inputs:
    targetType: 'inline'
    script: |
      python -m venv anass
      source anass/bin/activate
      python --version

# Instalar las dependencias en el entorno virtual 
- task: Bash@3
  displayName: 'Instalar las dependencias en el entorno virtual '
  inputs:
    targetType: 'inline'
    script: |
      source anass/bin/activate
      pip install --upgrade pip
      mkdir -p $(Build.ArtifactStagingDirectory)/dependencies
      pip install -r requirements.txt -t $(Build.ArtifactStagingDirectory)/dependencies
      pip install bandit pip-audit

# Ejecutar pip-audit para mostrar las vulnerabilidades
- task: Bash@3
  displayName: 'pip-audit scan'
  inputs:
    targetType: 'inline'
    script: |
      source anass/bin/activate
      pip-audit
  continueOnError: true

# Instalar Zip
- task: Bash@3
  displayName: 'Instalar zip'
  inputs:
    targetType: 'inline'
    script: |
      sudo apt-get install zip

# Zipear la carpeta del entorno virtual y el codigo python
- task: Bash@3
  displayName: 'Comprimir el entorno virtual y el codigo python'
  inputs:
    targetType: 'inline'
    script: |
      # Create directory for the artifact
      mkdir -p $(Build.ArtifactStagingDirectory)/vulnerabilities
      cp -r $(Build.ArtifactStagingDirectory)/dependencies $(Build.ArtifactStagingDirectory)/vulnerabilities
      cp -r *.txt $(Build.ArtifactStagingDirectory)/vulnerabilities
      cp -r *.py $(Build.ArtifactStagingDirectory)/vulnerabilities
      cd $(Build.ArtifactStagingDirectory)
      zip -r anass.zip vulnerabilities
      echo "Folder compressed successfully"

# Publicar el artefacto
- task: PublishBuildArtifacts@1
  displayName: 'Publicar el artefacto'
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)/anass.zip'
    ArtifactName: 'anass'
    publishLocation: 'Container'

```

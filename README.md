# Automatic Deployment of .NET Applications to IIS with Github Self-Hosted Runners
![image](https://github.com/sagarshrestha24/simple-web-app-mvc-dotnet/assets/76894861/7ead22fd-8b65-4998-aebb-c3efe6dbcd08)

###  How to setup Github Self-Hosted Runners to automatically build, publish and deploy your .NET Applications to an IIS

At work we've got a single Windows Server that hosts our websites using IIS. It has more than enough power to build our .NET projects alongside hosting and would free up time manually having to build projects and publish them to the IIS, whenever finished features are pushed to our  branch.

What I came up with was to use a GitHub Self-Hosted Actions Runner on the Windows Server, which would build, publish and restart the server automatically, whenever we push changes to the  branch.

### Installing the Actions Runner instance

Navigate to your GitHub organization's settings page, then navigate to Actions > Runners.

Then click on New Runner > New self-hosted runner as seen below.
![image](https://github.com/sagarshrestha24/simple-web-app-mvc-dotnet/assets/76894861/7c2a3b39-3b59-4a3c-bd19-8b5df80fe978)

Choose Windows and follow the installations options on the page.

When running the configuration, you will be promptet to setup a Windows Service, choose yes. When asked what user should be running the service set it to NT AUTHORITY\SYSTEM, this can be a huge security risk, only do this if you're aware of the risks. It is however required to be able to start and stop your website when publishing.

### Actions configuration

Now that the Actions Runner service is setup and running, we'll have to configure our .NET repository with a GitHub action.

Start by creating a file in your repository at the following directory .github/workflows/action.yml you can change the name of the file to whatever you'd like.

Now let's edit the action like the following

```
name: .NET

on:
  push:
    branches: [ "master" ]

jobs:
  build:

    runs-on: self-hosted

    steps:
    - uses: actions/checkout@v4
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: 7.0.x
    - name: Restore dependencies
      run: dotnet restore
    - name: Build
      run: dotnet build --no-restore
    - name: dotnet publish
      run: |
        dotnet build 
        dotnet publish -c Release -o .\test
    - name: Deploy to IIS
      run: |
         iisreset /stop
         xcopy /s /y .\test\* C:\Users\Administrator\Documents\IISExpress\WebSite\test
         iisreset /start
```

What this configuration does, is that it listens on the master branch, whenever changes are pushed it will start our self-hosted runner and begin the build process using cmd as the preffered shell as I've previously had issues with using Windows Powershell and Powershell 7.

The build steps starts off by running a git checkout followed by a restoring, building and testing the project. Once that's all done it will stop our website (in this case example.com) to ensure that the following publish will work without issues.




# DevOps Roadshow
This is a DevOps MVP based around an example ASP.NET Core web app. The webapp was generated by `dotnet new mvc`

Please clone this repo with **SSH Keys**, here are some [good instructions](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#platform-windows).

## Development environments
Setup documented below for Windows(Local and Cloud), and Mac/OSX.

### Windows Local 
First install the [latest .NET Core SDK](https://www.microsoft.com/net/core#windowscmd) TODO: automate this upgrade

```ps1
cd aspnetapp
# rd obj #to remove cache
dotnet restore
dotnet run
```

your app is running on `http://localhost:5000`

Windows / virtualization notes:
- Virtualizaiton still seems young on the Windows platform.
- Windows does not support nested virtualization -- i.e. Docker running on Windows that is already under virtualization.
- Windows Docker containers like `windows/nanoserver` only run on Windows.
- Azure has some support for nested virtualization ("Server2016 with Containers", running on special hardware) but it seems buggy.
- ATM, docker-compose doesn't work on Server2016
- ATM, Server2016 has a bug that prevents connecting to Docker containers on localhost

### OSX Local
On OSX we use a Docker container to run our ASP.Net app.  [Make sure "Docker for Mac" is installed](https://docs.docker.com/docker-for-mac/install/#download-docker-for-mac)

```sh
docker build -f Dockerfile.mac -t aspnetapp .
docker run -p 5000:5000 aspnetapp
```

Your app is running on `http://localhost:5000`.  Note that your `Project.cs` must be listening on `http://*:5000/`. 

Debug container build:
```sh
docker ps
docker run -it <container-id> sh
```

### Azure DevTestLabs
After investigation, [Azure DevTestLabs](https://azure.microsoft.com/en-us/services/devtest-lab/) seem like a good product, we can:
- automate creation of DTL VMs with Windows or Linux
- automate provisioning of DTL VMs with things like git, VSCode, etc.
- automate shutdown of these VMs at the end of a workday to save money
- DTL VMs can be provisioned on "Server2016 with Containers" for potential future Dockering
- Users connect to DTL VMs via RDP
- There is VSTS/DTL integration that I didn't investigate

A rough estimate puts cost of this service at **$50/developer/month**.
Leaving DTL notes here. We are not using DTL right now -- it doesn't give us better virtualization and they cost money.

## Staging environments

We plan to highlight a couple options for cloud-based staging environments.

### Cloud.gov
[Cloud.gov](https://cloud.gov) is Platform as a Service(PaaS) running on AWS and operated by 18F.  It can run .NET Core appliations.
Unfortunately cloud.gov only services federal customers right now. We use it as a free stand-in for other PaaS choices.

First, [Get a cloud.gov](https://cloud.gov/docs/getting-started/accounts/) account and [Set up the CLI](https://cloud.gov/docs/getting-started/setup/)

```
# Set up
cf login -a api.fr.cloud.gov --sso
cf target -o sandbox-gsa -s clinton.troxel

# Deploy
cd  aspnetapp
cf push aspnet-clint

# Get more information
cf apps
cf logs aspnet-clint --recent
open https://aspnet-clint.app.cloud.gov
```

Note: with default setup, we need to comment out `//.UseUrls("http://*:5000/")` in order for this app to run under cloud.gov.  This setting was added to support Docker.

## Azure

We demonstrate the sample app running on a VM in Microsoft Azure.

### Permissions needed in order to create infrastructure

Adding resources to Azure requires "Contributor" permissions.  This can be granted in two ways:
- At the Azure subscription level
- At the Azure resource group level 

####Add an Azure subcription contributor
1.  Select _More services >_ from the bottom of the Azure Portal services menu (the services list panel opens)
2.  Search for "Subscriptions" and click the Subscriptions tile (the Subscriptions list panel opens)
3.  Click on the subscription you want to add a new contributor to (the management panel for that subscription opens)
4.  Click on _Access control (IAM)_ (the access management panel opens)
5.  Click on the _Roles_ toolbar button (the roles panel opens)
6.  Click on the _Contributor_ role (the contributor group membership panel opens)
7.  Add the user to the group and close the panels

####Add an Azure resource group contributor
1.  Select _Resource groups_ from the Azure Portal services menu
2.  Click on the resource group you want to add a new contributor to (the management panel for that resource group opens)
3.  _Continue as from step 4 of the procedure for adding an Azure subscription contributor_

# Cognitive Factory : create the initial projects

Procedure last updated : 23-09-2018

**NOTE : YOU DON'T NEED TO DO THIS ANYMORE, THE PROJECTS HAVE BEEN INITIALIZED ONCE AND FOR ALL IN GITHUB**

**YOU CAN JUST CLONE THE EXISTING GITHUB REPOSITORY AT THE BEGINNING OF STEP 3**

## Initialize ASP.Net Core API and MySQL data persistence

1.  Create a Github repository
- login to the Github web site
- navigate the cognitivefactory organization
- click on the button to create a new repository : "cognitivefactory-platform"
- include a README.md, no license, a .gitignore file for Visual Studio
- create and commit a LICENSE.md file with the raw content of the CeCILL-C license

2. Clone the newly created Github repository in a local directory
- Open Visual Studio 2017 preview
- display Team Explorer window
- click on the plug icon : "Manage connections"
- connect to your Github account (member of the congitivefactory organization)
- clone the (almost) empty cognitivefactory-platform project in a local directory

3. Create an empty Visual Studio solution in this directory
- file / new project
- select template : Other project types / Visual Studio Solutions
- name : cognitivefactory-platform
- select the local directory of the Github repo cloned just above
- don't create a new solution directory, don't create a new Git repository

4. Create an ASP.Net Core 2.2 API project
- file / new project
- select template : Web / ASP.Net Core
- name : CognitiveFactory.Platform.API
- select framweork version : ASP.Net Core 2.2
- select template : API
- hit F5, accept to install the self-signed certificate
- check that the values are returned
- stop the webserver

5. Scaffold a first API to manage platform solutions
- delete ValuesController.cs
- create a Models directory
- add a new Solution class to the model directory
- create 3 properies on this class : int Id, string Name, string Description
- right-click on the Controllers directory : add / Controller
- select template : API Controller with actions, using Entity Framework
- select model class : Solution
- data context class : click on + to create a new one, CognitiveFactory.Platform.API.Data.CognitiveFactoryPlatformAPIContext
- open Data/CognitiveFactoryPlatformAPIContext.cs : rename Solution property to Solutions

5. Initialize the MySQL persistence
- right-click on the the project / Manage Nuget packages
- find and install the package : Pomelo.EntityFrameworkCore.MySql [v2.1.2]
- TO DO : find how to change the default character set of the MySQL database instance to "utf8mb4" (no env variable in Openshift image, it seems that my.cnf config file needs to be changed and server restarted)
- open class Startup.cs, in ConfigureServices method add :

using Pomelo.EntityFrameworkCore.MySql.Infrastructure;

- remove :

options.UseSqlServer ...

- and replace with :

options => options.UseMySql(
    Configuration.GetConnectionString("CognitiveFactoryPlatformAPIContext"),
    mysqlOptions => { mysqlOptions.ServerVersion(new Version(5, 7, 21), ServerType.MySql); } )

- in appsettings.json, replace the following value : 

"CognitiveFactoryPlatformAPIContext": "Server=openshift.local;Port=32396;Database=cognitivefactorydb;User=mysqluser;Password=mysqluserpassword;"

- open the Package Manager Console (display / windows / others)
- type : Add-Migration Initial -OutputDir Data/Migrations
- type : Update-Database

6. Test the API
- edit file Properties/launchSettings.json : replace twice "launchUrl": "api/values" by "launchUrl": "api/solutions",
- type F5 to launch the project in debug mode
- you should see [] in your browser
- open Postman
- GET https://localhost:44391/api/solutions / launch / save as "Get solutions"
- POST https://localhost:44391/api/solutions, body application/json = {	Name: "Solution1", Description:"First solution" }, / launch / save as "Create solution"
- Get solutions again

7. Activate and enhance Swagger documentation


## Package the ASP.Net Core API as a docker container

0. Understand how OKD builds work at : 
- [https://docs.okd.io/3.10/dev_guide/builds/index.html](https://docs.okd.io/3.10/dev_guide/builds/index.html).
- [https://docs.okd.io/3.10/dev_guide/managing_images.html](https://docs.okd.io/3.10/dev_guide/managing_images.html)
- [https://docs.okd.io/latest/dev_guide/dev_tutorials/binary_builds.html](https://docs.okd.io/latest/dev_guide/dev_tutorials/binary_builds.html)

1. Create a Dockerfile at the root of the Visual Studio solution

```Dockerfile
FROM microsoft/dotnet:2.2-sdk-alpine AS build
WORKDIR /app

# copy csproj and restore as distinct layers
COPY *.sln .
COPY CognitiveFactory.Platform.API/*.csproj ./CognitiveFactory.Platform.API/
RUN dotnet restore

# copy everything else and build app
COPY CognitiveFactory.Platform.API/. ./CognitiveFactory.Platform.API/
WORKDIR /app/CognitiveFactory.Platform.API
RUN dotnet publish -c Release -o out


FROM microsoft/dotnet:2.2-aspnetcore-runtime-alpine AS runtime
WORKDIR /app
COPY --from=build /app/CognitiveFactory.Platform.API/out ./
ENTRYPOINT ["dotnet", "CognitiveFactory.Platform.API.dll"]'
````

2. Declare a new build in the current Openshift project
> oc new-build --binary=true --name=cognitivefactory-api-build --strategy=docker --to=cognitivefactory-api

3. Start a new build from the solution directory
> oc start-build cognitivefactory-api-build --follow=true --from-dir=C:\Users\laure\OneDrive\Dev\Github\cognitivefactory\cognitivefactory-platform

4. Start a new service from this image
> oc new-app cognitivefactory-api
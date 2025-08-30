---
title: "dotnet error: Unable to read a package reference from the project"
seoTitle: "dotnet error: Unable to read a package reference from the project"
seoDescription: "cannot find the list of packages installed in dotnet by running: dotnet list package. it says "Unable to read a package reference from the project""
datePublished: Mon Aug 25 2025 12:42:53 GMT+0000 (Coordinated Universal Time)
cuid: cmer3xzev000302jr0q7ib86o
slug: dotnet-error-unable-to-read-a-package-reference-from-the-project
tags: dotnetcore, web-api, dot-net-development, dotnet-cli, dotnet-error

---

when you try to view all the packages installed in your project by using the command:

```bash
dotnet list package
```

and get the following error:

```bash
Unable to read a package reference from the project `C:\reallyLongListOfDirectories\jwt_auth\jwt_auth.csproj`.Please make sure that your project file and project.assets.json file are in sync by running restore.
```

Solution:

simply delete the file:  
obj&gt;project.assests.json

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756125191687/bd13d3dd-c197-4de4-9f8e-868956596aed.png align="center")

after deleting the file run the following command:

```bash
dotnet restore
```

```bash
dotnet list package
```

This usually solves the problem, in case it doesn’t work. Please make sure of the following:

1. Your local server should NOT be running.
    
2. Your project is configured properly.
    
3. If a package is being added, it can cause the error wait for a moment and try again.
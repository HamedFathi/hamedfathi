---
title: A Professional ASP.NET Core - File Upload
date: October 9 2020
category: aspnetcore
tags:
    - dotnet
    - aspnetcore
    - file
    - upload
---

ASP.NET Core supports uploading one or more files using buffered model binding for smaller files and unbuffered streaming for larger files.

<!-- more -->

## File Model

Create a new class, `Models/FileModel.cs`. This will be the base class.

```cs
public abstract class FileModel
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string FileType { get; set; }
    public string Extension { get; set; }
    public string Description { get; set; }
    public string UploadedBy { get; set; }
    public DateTime? CreatedOn { get; set; }
}
```

Now, let's create a model for the file on the file system. Name it `Models/FileOnFileSystem.cs` and inherit the `FileModel` class.

```cs
public class FileOnFileSystemModel : FileModel
{
    public string FilePath { get; set; }
}
```

Similarly add another class for the file on database, `Models/FileOnDatabaseModel.cs`

```cs
public class FileOnDatabaseModel : FileModel
{
    public byte[] Data { get; set; }
}
```

## Setting up Entity Framework Core

Install the below packages

```bash
Install-Package Microsoft.EntityFrameworkCore -Version 3.1.9
dotnet add package Microsoft.EntityFrameworkCore --version 3.1.9
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="3.1.9" />

Install-Package Microsoft.EntityFrameworkCore.Design -Version 3.1.9
dotnet add package Microsoft.EntityFrameworkCore.Design --version 3.1.9
<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="3.1.9">
  <PrivateAssets>all</PrivateAssets>
  <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
</PackageReference>

Install-Package Microsoft.EntityFrameworkCore.Tools -Version 3.1.9
dotnet add package Microsoft.EntityFrameworkCore.Tools --version 3.1.9
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="3.1.9">
  <PrivateAssets>all</PrivateAssets>
  <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
</PackageReference>

Install-Package Microsoft.EntityFrameworkCore.SqlServer -Version 3.1.9
dotnet add package Microsoft.EntityFrameworkCore.SqlServer --version 3.1.9
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="3.1.9" />
```

Next, add a connection string to your `appsetting.json` file.

```json
"ConnectionStrings": {
  "DefaultConnection": "Server=.;Database=FileDb;Trusted_Connection=True;"
}
```

Create `ApplicationDbContext`

```cs
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    { }
    public DbSet<FileOnFileSystemModel> FileOnFileSystemModel { get; set; }
    public DbSet<FileOnDatabaseModel> FileOnDatabaseModel { get; set; }
}
```

Let's now configure the services. Modify the `Startup.cs/ConfigureServices`.

```cs
using Microsoft.EntityFrameworkCore;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // ...
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(
            Configuration.GetConnectionString("DefaultConnection"),
            b => b.MigrationsAssembly(typeof(ApplicationDbContext).Assembly.FullName))
        );
        // ...
    }
}
```

Finally, let's do the required migrations and update our database. Just run the following commands on the `Package Manager Console`.

```bash
Add-Migration initial
Update-Database

// dotnet tool install --global dotnet-ef
// dotnet tool update --global dotnet-ef
dotnet ef migrations add initial
dotnet ef database update
```

You will get a done message on console. Open up SQL Server Object Explorer to check if the database and tables have been created.

![](/images/a-professional-asp.net-core-file-upload/db.png)

## Setting up the View and ViewModel

Make a new class, a ViewModel class, `Models/FileUploadViewModel.cs` as below.

```cs
public class FileUploadViewModel
{
    public List<FileOnFileSystemModel> FilesOnFileSystem { get; set; }
    public List<FileOnDatabaseModel> FilesOnDatabase { get; set; }
}
```

After that, Let's start modifying the View Page, `Views/File/Index.cshtml`.

```html
@model FileUploadViewModel
@{
    ViewData["Title"] = "Index";
    Layout = "~/Views/Shared/_Layout.cshtml";
}
<h4>Start Uploading Files Here</h4>
<hr />
@if (ViewBag.Message != null)
{
    <div class="alert alert-success alert-dismissible" style="margin-top:20px">
        @ViewBag.Message
    </div>
}
<form method="post" enctype="multipart/form-data">
    <input type="file" name="files" multiple required />
    <input type="text" autocomplete="off" placeholder="Enter File Description" name="description" required />
    <button type="submit" class="btn btn-primary" asp-controller="File" asp-action="UploadToFileSystem">Upload to File System</button>
    <button class="btn btn-success" type="submit" asp-controller="File" asp-action="UploadToDatabase">Upload to Database</button>
</form>
```


















## Reference(s)

Most of the information in this article has gathered from various references.

* https://code-maze.com/file-upload-aspnetcore-mvc/
* https://khalidabuhakmeh.com/upload-a-file-using-aspdotnet-core
* https://www.c-sharpcorner.com/article/upload-download-files-in-asp-net-core-2-0
* https://gunnarpeipman.com/aspnet-core-file-uploads/
* https://www.codewithmukesh.com/blog/file-upload-in-aspnet-core-mvc/
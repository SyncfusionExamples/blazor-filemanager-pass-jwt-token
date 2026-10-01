# Blazor-filemanager-pass-jwt-token

**Repository Description**  
This repository contains a **Blazor File Manager** sample that demonstrates how to **send a JWT token (authorization header)** from the client side to the server when performing File Manager operations using the [Blazor File Manager](https://www.syncfusion.com/blazor-components/blazor-file-manager) component.

The sample explains how authorization data can be passed for **read**, **upload**, **download**, and **image preview** operations using Blazor events and custom server‑side handling.

## Project Overview

The purpose of this project is to help developers understand how to secure File Manager operations in a Blazor application by attaching custom authorization values (such as JWT tokens) to requests sent from the client to the server.

It shows how different File Manager events can be leveraged to inject authorization headers or custom parameters for various file operations.

## Key Features

- Uses `SfFileManager` to display and manage files and folders.
- Uses `FileManagerEvents<FileManagerDirectoryContent>` to intercept requests.
- Implements the `OnSend` event to add authorization data through `args.HttpClientInstance.DefaultRequestHeaders`.
- Uses `BeforeDownload` to cancel the default download behavior with `args.Cancel = true` and trigger a custom download flow through JavaScript interop.
- Uses `BeforeImageLoad` to append custom values to `args.ImageUrl`.
- Reads authorization values from `HttpContext.Request.Headers["Authorization"]` in controller actions.
- Uses `PhysicalFileProvider` from `Syncfusion.EJ2.FileManager.PhysicalFileProvider` to perform file operations.

## Prerequisites

Ensure the following requirements are met before running this project:
- [Visual Studio 2022](https://visualstudio.microsoft.com/vs/) or [Visual Studio Code](https://code.visualstudio.com/)
- [.NET SDK](https://dotnet.microsoft.com/en-us/download/dotnet) compatible with the project's target framework
- ASP.NET Core Blazor Server support
- Syncfusion Blazor components installed

## How to Run the Project

**Visual Studio 2022**

1. Clone or download the repository.
2. Open `JwtTokenSample/TwtTokenSample.sln`.
3. Restore NuGet packages.
4. Build the solution.
5. Ensure the `TwtTokenSample` project is the startup project.
6. Run the application using `Ctrl+F5`.

**Visual Studio Code**

1. Open the repository folder.
2. Navigate to the project directory.

```bash
cd JwtTokenSample
dotnet restore
dotnet run
```

3. Open the URL displayed in the terminal after the application starts.

## Usage

### File Manager authorization header for read and upload operation

To send the authorization header data from client side to server side use the FileManager [`Onsend`](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.FileManager.FileManagerEvents-1.html?_ga=2.138704048.2095323515.1658726624-1114855823.1658293399#Syncfusion_Blazor_FileManager_FileManagerEvents_1_OnSend) event. 

Refer to the code snippet of `Index.razor` page
```
<SfFileManager TValue="FileManagerDirectoryContent">
...
<FileManagerEvents TValue="FileManagerDirectoryContent" OnSend="onsend" BeforeDownload="beforeDownload" BeforeImageLoad="beforeImageLoad"></FileManagerEvents>
</SfFileManager>
@code{
    void onsend(BeforeSendEventArgs args)
    {
        if (isRead && args.Action=="read")
        {
            args.HttpClientInstance.DefaultRequestHeaders.Add("Authorization", "Documents");
            isRead = false;
        }
    }
}
```
Refer to the code snippet of `FileManagerController.cs` page
```
public object FileOperations([FromBody] FileManagerDirectoryContent args)
{
    var root = HttpContext.Request.Headers["Authorization"];
    ...
}
...
public IActionResult Upload(string path, IList<IFormFile> uploadFiles, string action, string data)
{
    var root = HttpContext.Request.Headers["Authorization"].ToString().Split(',')[0];
    ...
}
```
### File Manager authorization header for Download operation

Since there is no direct way to pass custom value, you can prevent our default download operation by setting `args.Cancel` as true in [`BeforeDownload`](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.FileManager.FileManagerEvents-1.html#Syncfusion_Blazor_FileManager_FileManagerEvents_1_BeforeDownload)
event. Then you can trigger the customized download operation using an interop call where you can pass custom values to server side. Check out the below code snippet.

Refer to the code snippet of `Index.razor` page
```
@inject IJSRuntime jsRuntime
<SfFileManager TValue="FileManagerDirectoryContent">
...
<FileManagerEvents TValue="FileManagerDirectoryContent" OnSend="onsend" BeforeDownload="beforeDownload" BeforeImageLoad="beforeImageLoad"></FileManagerEvents>
</SfFileManager>
@code{
public async Task beforeDownload(BeforeDownloadEventArgs<FileManagerDirectoryContent> args)
{
    args.Cancel = true;
    DirectoryContent[] data = new DirectoryContent[]{ new DirectoryContent()
    {
        Name = args.Data.DownloadFileDetails[0].Name, // name of the file
        IsFile = args.Data.DownloadFileDetails[0].IsFile, // indicates whether file or folder
        FilterPath = args.Data.DownloadFileDetails[0].FilterPath, // path of the file/folder from root directory
        HasChild = args.Data.DownloadFileDetails[0].HasChild, // if folder has child folder set as true else false
        Type = args.Data.DownloadFileDetails[0].Type // empty string for folder and file type like .png for files 
    } };
    DownloadUrl = https://localhost:44355/api/FileManager/Download;
    DirectoryContent downloadData = new DirectoryContent()
    {
        Data = data,
        Path = args.Data.Path,// path in which the file is located (make ensure to add the path from root directory excluding the root directory name)
        Names = args.Data.Names// names of the files to be downloaded in the specified path
    }; 
    await jsRuntime.InvokeAsync<object>("saveFile", downloadData, DownloadUrl);
}
}
```
Refer to the code snippet of `_Host.cshtml` page
```
<script>  
    window.saveFile = (data, downloadUrl) => {  
        //creating the data to call download web API method 
        var i = {
            action: "download", 
            ath: data.path,  
            names: data.names, 
            data: data.data, 
            customvalue: "Pictures",  
        }   
... 
//appeding the dynamically created form to the document and perform form submit to perform download operation   
    a.appendChild(s),  
        document.body.appendChild(a),
        document.forms.namedItem("downloadForm").submit(), 
        document.body.removeChild(a) 
    } 
</script>
```
Refer to the code snippet of `FileManagerController.cs` page

```
[Route("Download")]   
public IActionResult Download(string downloadInput)   
{   
    FileManagerDirectoryContentExtend args = JsonConvert.DeserializeObject<FileManagerDirectoryContentExtend>(downloadInput);   
    var root = args.customvalue;   
    this.operation.RootFolder(this.basePath + "\\" + this.root + "\\" + root);
...
```
### File Manager authorization header for GetImage operation

There is no direct support to pass header in GetImage operation. However, you can pass the custom data in the imageUrl, but this is not preferable for sensitive data sending.

Refer to the code snippet of `Index.razor` page
```
<SfFileManager TValue="FileManagerDirectoryContent">
...
<FileManagerEvents TValue="FileManagerDirectoryContent" OnSend="onsend" BeforeDownload="beforeDownload" BeforeImageLoad="beforeImageLoad"></FileManagerEvents>
</SfFileManager>
@code{
    public void beforeImageLoad(BeforeImageLoadEventArgs<FileManagerDirectoryContent> args)
    {
        args.ImageUrl = args.ImageUrl + "&SubFolder=Pictures";
    }
}
```
Refer to the code snippet of `FileManagerController.cs` page
```
public class FileManagerDirectoryContentExtend : FileManagerDirectoryContent    
{    
    public string customvalue { get; set; }    
    public string SubFolder { get; set; }
}    
...
[Route("GetImage")]
public IActionResult GetImage(FileManagerDirectoryContentExtend args)
{
    var root = args.SubFolder;
    this.operation.RootFolder(this.basePath + "\\" + this.root + "\\" + root);
    return this.operation.GetImage(args.Path, args.Id, false, null, null);
}
```

## Configuration

Authorization handling is configured using:
- Blazor File Manager events on the client
- Custom ASP.NET Core controller methods on the server
- Optional JavaScript interop for download operations

## Project Structure

`JwtTokenSample/Pages/Index.razor` - Main sample page hosting the Syncfusion Blazor File Manager component. Configures `FileManagerAjaxSettings` and handles the `OnSend`, `BeforeDownload`, and `BeforeImageLoad` events to transmit authorization data during read, upload, download, and image retrieval operations.

`JwtTokenSample/Controllers/FileManagerController.cs` - Server-side File Manager controller implementing `FileOperations`, `Upload`, `Download`, and `GetImage` endpoints. Reads authorization values from request headers and custom parameters, configures the `PhysicalFileProvider` root folder dynamically, and performs CRUD file system operations.

`JwtTokenSample/wwwroot/Files/` - Physical file storage location used by the `PhysicalFileProvider`. Subfolders are dynamically selected based on authorization values received from the client.

`JwtTokenSample/wwwroot/` - Static web assets, scripts, and resources used by the Blazor application and custom File Manager download functionality.

## Support and Feedback

- For general product questions, visit the [Syncfusion Community Forum](https://www.syncfusion.com/forums) or [Syncfusion Support](https://www.syncfusion.com/support).
- To report an issue specific to this sample, open a GitHub issue in this repository.
- Refer to the Syncfusion File Manager documentation: https://help.syncfusion.com/file-manager-sdk/blazor/overview

## License

This is a Syncfusion sample project provided to demonstrate product usage. Review the [Syncfusion license terms](https://www.syncfusion.com/sales/pricing?category=ui-components) before using Syncfusion components in your own applications.

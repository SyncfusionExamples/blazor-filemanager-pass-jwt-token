# Blazor-filemanager-pass-jwt-token

**Repository Description**  
This repository contains a **Blazor File Manager** sample that demonstrates how to **send a JWT token (authorization header)** from the client side to the server when performing File Manager operations using the [Blazor File Manager](https://www.syncfusion.com/blazor-components/blazor-file-manager) component.

The sample explains how authorization data can be passed for **read**, **upload**, **download**, and **image preview** operations using Blazor events and custom server‑side handling.

## Project Overview
The purpose of this project is to help developers understand how to secure File Manager operations in a Blazor application by attaching custom authorization values (such as JWT tokens) to requests sent from the client to the server.

It shows how different File Manager events can be leveraged to inject authorization headers or custom parameters for various file operations.

## Features
- Integration of **Blazor File Manager**
- Pass JWT or custom authorization headers from client to server
- Handle authorization for **read**, **upload**, **download**, and **image preview** operations
- Use File Manager events such as `OnSend`, `BeforeDownload`, and `BeforeImageLoad`
- Secure server‑side request handling in ASP.NET Core controllers

## Prerequisites
Ensure the following requirements are met before running this project:
- Visual Studio 2022  
- .NET SDK compatible with Blazor  
- Basic knowledge of Blazor and ASP.NET Core  
- Syncfusion Blazor components installed

## Installation and Running the Project

* Checkout this project to a location in your disk.
* Open the solution file using the Visual Studio 2022.
* Restore the NuGet packages by rebuilding the solution.
* Run the project.

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

## Documentation
- General Syncfusion documentation:
https://help.syncfusion.com/
- Blazor Introduction:
https://blazor.syncfusion.com/documentation/introduction
- Blazor File Manager – Getting Started:
https://blazor.syncfusion.com/documentation/file-manager/getting-started-with-web-app

## Additional Resources
- Blazor File Manager product overview:
https://www.syncfusion.com/blazor-components/blazor-file-manager

## Troubleshooting
- Ensure the OnSend event is triggered for read and upload operations.
- Verify JavaScript interop is correctly registered for custom downloads.
- Validate server‑side controller methods receive expected parameters.
- Rebuild and restart the application after configuration changes.

## Support
For detailed API references, security customization guidance, and advanced Blazor File Manager scenarios, refer to the Syncfusion Blazor documentation links above.

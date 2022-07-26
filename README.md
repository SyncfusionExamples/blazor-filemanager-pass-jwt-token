# blazor-filemanager-pass-jwt-token

This repository contains the Blazor FileManager component with HttpClient to get data from server in the Blazor File Manager component.

## How to run this application?

To run this application, you need to first clone the [`blazor-filemanager-pass-jwt-token`](https://github.com/SyncfusionExamples/blazor-filemanager-pass-jwt-token) repository and then navigate to its appropriate path where it has been located in your system.

To do so, open the command prompt and run the below commands one after the other.

```
git clone https://github.com/SyncfusionExamples/blazor-filemanager-pass-jwt-token 

cd blazor-filemanager-pass-jwt-token

```

## Restore the NuGet package and build the application

To restore the NuGet package, run the following command in root folder of the application.

```
dotnet restore
```

To build the application, run the following command.

```
dotnet build
```

## Running application

After successful compilation, run the following command to run the application.

```
dotnet run
```

## File Manager authorization header for read and upload operation

To send the authorization header data from client side to server side using the FileManager [`Onsend`](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.FileManager.FileManagerEvents-1.html?_ga=2.138704048.2095323515.1658726624-1114855823.1658293399#Syncfusion_Blazor_FileManager_FileManagerEvents_1_OnSend) event. To send the header data to server side just map the following code snippet in the `Index.razor` page and `FileManagerController.cs` page property of File Manager.

Refer the code snippet of `Index.razor` page

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

Refer the code snippet of `FileManagerController.cs` page

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

## File Manager authorization header for Download operation

Since there is no direct way to pass custom value, you can prevent our default download operation by setting `args.Cancel` as true in [`BeforeDownload`](https://help.syncfusion.com/cr/blazor/Syncfusion.Blazor.FileManager.FileManagerEvents-1.html#Syncfusion_Blazor_FileManager_FileManagerEvents_1_BeforeDownload)
event. Then you can trigger the customized download operation using an interop call where you can pass custom values to server side. Check out the below code snippet.

Refer the code snippet of `Index.razor` page

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

Refer the code snippet of `_Host.cshtml` page

```
<script>  
        window.saveFile = (data, downloadUrl) => {  
            //creating the data to call download web API method 
            var i = {
                action: "download", 
                path: data.path,  
                names: data.names, 
                data: data.data, 
                customvalue: "Pictures",  
            }   
            …  
//appeding the dynamically created form to the document and perform form submit to perform download operation   
            a.appendChild(s),  
                document.body.appendChild(a),
                document.forms.namedItem("downloadForm").submit(), 
                document.body.removeChild(a) 
        } 
</script>
```

## File Manager authorization header for GetImage operation

We are unable to add the custom header in GetImage request since the GetImage is processed using query string parameter and download processed using form element.

If we are going to use ajax (asynchronous request) then, each image creates individual request and its having certain time delay based on image size, network bandwidth and server respond time. We are unable to update the headers to the corresponding image tag value. So, it is not possible to implement these operations using Ajax requests. However, you can pass the custom data in the imageUrl, but this is not preferable for sensitive data sending.

Refer the code snippet of `Index.razor` page

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

Refer the code snippet of `FileManagerController.cs` page

```
public class FileManagerDirectoryContentExtend : FileManagerDirectoryContent    
{    
    public string customvalue { get; set; }    
    public string SubFolder { get; set; }
}    
...
[Route("Download")]   
public IActionResult Download(string downloadInput)   
{   
            FileManagerDirectoryContentExtend args = JsonConvert.DeserializeObject<FileManagerDirectoryContentExtend>(downloadInput);   
            var root = args.customvalue;   
              this.operation.RootFolder(this.basePath + "\\" + this.root + "\\" + root);
…
[Route("GetImage")]
public IActionResult GetImage(FileManagerDirectoryContentExtend args)
{
    var root = args.SubFolder;
    this.operation.RootFolder(this.basePath + "\\" + this.root + "\\" + root);
    return this.operation.GetImage(args.Path, args.Id, false, null, null);
}
```
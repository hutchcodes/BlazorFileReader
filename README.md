[![Build status](https://ci.appveyor.com/api/projects/status/rr7pchwk7wbc3mn1/branch/master?svg=true)](https://ci.appveyor.com/project/Tewr/blazorfilereader/branch/master)
[![NuGet](https://img.shields.io/nuget/v/Tewr.Blazor.FileReader.svg)](https://www.nuget.org/packages/Tewr.Blazor.FileReader)

# BlazorFileReader
Blazor library and Demo of read-only file streams in [Blazor](https://github.com/aspnet/Blazor). Originally built for Wasm ("Client-side" Blazor), Server-side Blazor is also supported as of version 0.7.1.

This library exposes read-only streams using ```<input type="file" />```
and [FileReader](https://developer.mozilla.org/en-US/docs/Web/API/FileReader).

Here is a [Live demo](https://tewr.github.io/BlazorFileReader/) that contains the output of [the wasm demo project](src/Blazor.FileReader.Wasm.Demo). Currently, its a build based on ```v0.5.1```.

## Installation

Use [Nuget](https://www.nuget.org/packages/Tewr.Blazor.FileReader): ```Install-Package Tewr.Blazor.FileReader```

## Usage

Depending on your [project type](https://docs.microsoft.com/en-us/aspnet/core/razor-components/faq?view=aspnetcore-3.0), use one of the two examples below.

### Client-side / Wasm Project type
Setup IoC for ```IFileReaderService``` in ([Program.cs](src/Blazor.FileReader.Wasm.Demo/Program.cs#L24)):

```cs
   using Blazor.FileReader;
   static void Main(string[] args)
   {
       var serviceProvider = new BrowserServiceProvider(services =>
       {
            services.AddSingleton<IFileReaderService, FileReaderService>();
       });
       new BrowserRenderer(serviceProvider).AddComponent<App>("app");
       (...)
```

### Server-side / asp.net core Project type

Setup IoC for  ```IFileReaderService``` in ([Startup.cs](src/Blazor.FileReader.AspNetCore.Demo.App/Startup.cs#L10)):

```cs
using Blazor.FileReader;

    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton<IFileReaderService, FileReaderService>();
        }
	(...)
```

### Blazor View

The code for views looks the same for both client- and server-side projects

```cs
@page "/MyPage"
@using System.IO;
@inject IFileReaderService fileReaderService;

<input type="file" ref="inputTypeFileElement" /><button onclick="@ReadFile">Read file</button>

@functions {
    ElementRef inputTypeFileElement;

    public async Task ReadFile()
    {
        foreach (var file in await fileReaderService.CreateReference(inputTypeFileElement).EnumerateFilesAsync())
        {
            // Read into buffer and act (uses less memory)
            using(Stream stream = await file.OpenReadAsync()) {
                // Do stuff with stream...
                await stream.ReadAsync(buffer, ...);
		// This following will fail. Only async read is allowed.
		stream.Read(buffer, ...)
            }

            // Read into memory and act
            using(MemoryStream memoryStream = await file.CreateMemoryStreamAsync(4096)) {
	    	// Sync calls are ok once file is in memory
		memoryStream.Read(buffer, ...)
            }
        }
    }
}
```

## Notes

To use the code in this demo in your own project you need to use at least version 
```0.4.0``` of blazor (branch 0.4.0). 

The ```master``` branch uses ```0.7.1``` of Blazor.

Blazor is an experimental project, not ready for production use. Just as Blazor frequently has breaking changes, so does the API of this library.

### Version notes

Versions previous to ```0.7.1``` did not support server-side Blazor and would throw ```[System.PlatformNotSupportedException] Requires MonoWebAssemblyJSRuntime as the JSRuntime```.

Versions previous to ```0.5.1``` wrapped the input element in a Blazor Component, this has been removed for better configurability and general lack of value.



# AzureFunctionsAssemblyRepro

Reproduction steps:

1. Run the function.
2. Visit the [endpoint](http://localhost:7071/api/Function1).

To repro without cloning:

1. Create a new Azure Functions app (C#, .NET Core v3) with a single HTTP-triggered function (anonymous auth).
1. [Update the Functions SDK version to 3.0.5](https://github.com/StephenClearyExamples/AzureFunctionsAssemblyRepro/commit/8e4f8fd9b120f68fce1c6baa9a5f7c6a49e2665e).
1. [Add the `Microsoft.IdentityModel.Tokens` package and reference a type from that package in your function](https://github.com/StephenClearyExamples/AzureFunctionsAssemblyRepro/commit/d7f9d082d9ee8f97fdc9e90a5c3cc802fc5e107f).
1. Visit the [endpoint](http://localhost:7071/api/Function1).
1. Enjoy your `System.Private.CoreLib: Exception while executing function: Function1. FunctionApp1: Could not load file or assembly 'Microsoft.IdentityModel.Tokens, Version=6.5.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35'. The system cannot find the file specified.`
1. ???
1. Profit!

This likely happens with many other packages; I just happened to hit it with `Microsoft.IdentityModel.Tokens`.

## Details

I suspect the problem is caused by the `RemoveRuntimeDependencies` task from the `Microsoft.Azure.WebJobs.Script.ExtensionsMetadataGenerator` package. MSBuild logs seem to implicate it.

### Microsoft.NET.Sdk.Functions v3.0.5 (Microsoft.Azure.WebJobs.Script.ExtensionsMetadataGenerator v1.1.5)

Build succeeds, but most of the assemblies listed below are deleted from the output directory, causing runtime failure:

```
1>  Using "GenerateFunctions" task from assembly "C:\Users\steph\.nuget\packages\microsoft.net.sdk.functions\3.0.5\build\..\tools\netcoreapp3.0\\Microsoft.NET.Sdk.Functions.MSBuild.dll".
1>  Task "GenerateFunctions"
1>    Function generator path: 'dotnet'
1>    Microsoft.NET.Sdk.Functions.Generator.dll "C:\Work\AzureFunctionsAssemblyRepro\FunctionApp1\FunctionApp1\bin\Debug\netcoreapp3.1\bin\FunctionApp1.dll " "C:\Work\AzureFunctionsAssemblyRepro\FunctionApp1\FunctionApp1\bin\Debug\netcoreapp3.1\ " "False "
1>  Done executing task "GenerateFunctions".
1>Target _GenerateFunctionsExtensionsMetadataPostBuild:
1>  Using "GenerateFunctionsExtensionsMetadata" task from assembly "C:\Users\steph\.nuget\packages\microsoft.azure.webjobs.script.extensionsmetadatagenerator\1.1.5\build\..\tools\net46\Microsoft.Azure.WebJobs.Script.ExtensionsMetadataGenerator.dll".
1>  Task "GenerateFunctionsExtensionsMetadata"
1>    Extensions generator working directory: 'C:\Users\steph\.nuget\packages\microsoft.azure.webjobs.script.extensionsmetadatagenerator\1.1.5\tools\net46\..\netstandard2.0\generator'
1>    Extensions generator path: 'dotnet'
1>    Microsoft.Azure.WebJobs.Script.ExtensionsMetadataGenerator.Console.dll "C:\Work\AzureFunctionsAssemblyRepro\FunctionApp1\FunctionApp1\bin\Debug\netcoreapp3.1\bin" "C:\Work\AzureFunctionsAssemblyRepro\FunctionApp1\FunctionApp1\bin\Debug\netcoreapp3.1\bin\extensions.json"
1>    Found 52 assemblies to evaluate in 'C:\Work\AzureFunctionsAssemblyRepro\FunctionApp1\FunctionApp1\bin\Debug\netcoreapp3.1\bin':
1>      FunctionApp1.dll
1>        Resolving assembly: 'System.Runtime, Version=4.2.2.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a'
1>        Assembly 'System.Runtime' loaded.
1>      Microsoft.AspNetCore.Authentication.Abstractions.dll
1>      Microsoft.AspNetCore.Authentication.Core.dll
1>      Microsoft.AspNetCore.Authorization.dll
1>      Microsoft.AspNetCore.Authorization.Policy.dll
1>      Microsoft.AspNetCore.Hosting.Abstractions.dll
1>      Microsoft.AspNetCore.Hosting.Server.Abstractions.dll
1>      Microsoft.AspNetCore.Http.Abstractions.dll
1>      Microsoft.AspNetCore.Http.dll
1>      Microsoft.AspNetCore.Http.Extensions.dll
1>      Microsoft.AspNetCore.Http.Features.dll
1>      Microsoft.AspNetCore.JsonPatch.dll
1>      Microsoft.AspNetCore.Mvc.Abstractions.dll
1>      Microsoft.AspNetCore.Mvc.Core.dll
1>      Microsoft.AspNetCore.Mvc.Formatters.Json.dll
1>      Microsoft.AspNetCore.Mvc.WebApiCompatShim.dll
1>      Microsoft.AspNetCore.ResponseCaching.Abstractions.dll
1>      Microsoft.AspNetCore.Routing.Abstractions.dll
1>      Microsoft.AspNetCore.Routing.dll
1>      Microsoft.AspNetCore.WebUtilities.dll
1>      Microsoft.Azure.WebJobs.dll
1>      Microsoft.Azure.WebJobs.Host.dll
1>      Microsoft.Azure.WebJobs.Host.Storage.dll
1>      Microsoft.DotNet.PlatformAbstractions.dll
1>      Microsoft.Extensions.Configuration.Abstractions.dll
1>      Microsoft.Extensions.Configuration.Binder.dll
1>      Microsoft.Extensions.Configuration.dll
1>      Microsoft.Extensions.Configuration.EnvironmentVariables.dll
1>      Microsoft.Extensions.Configuration.FileExtensions.dll
1>      Microsoft.Extensions.Configuration.Json.dll
1>      Microsoft.Extensions.DependencyInjection.Abstractions.dll
1>      Microsoft.Extensions.DependencyInjection.dll
1>      Microsoft.Extensions.DependencyModel.dll
1>      Microsoft.Extensions.FileProviders.Abstractions.dll
1>      Microsoft.Extensions.FileProviders.Physical.dll
1>      Microsoft.Extensions.FileSystemGlobbing.dll
1>      Microsoft.Extensions.Hosting.Abstractions.dll
1>      Microsoft.Extensions.Hosting.dll
1>      Microsoft.Extensions.Logging.Abstractions.dll
1>      Microsoft.Extensions.Logging.Configuration.dll
1>      Microsoft.Extensions.Logging.dll
1>      Microsoft.Extensions.ObjectPool.dll
1>      Microsoft.Extensions.Options.ConfigurationExtensions.dll
1>      Microsoft.Extensions.Options.dll
1>      Microsoft.Extensions.Primitives.dll
1>      Microsoft.IdentityModel.Logging.dll
1>      Microsoft.IdentityModel.Tokens.dll
1>      Microsoft.Net.Http.Headers.dll
1>      Microsoft.WindowsAzure.Storage.dll
1>      NCrontab.Signed.dll
1>      Newtonsoft.Json.Bson.dll
1>      Newtonsoft.Json.dll
1>    'C:\Work\AzureFunctionsAssemblyRepro\FunctionApp1\FunctionApp1\bin\Debug\netcoreapp3.1\bin\extensions.json' successfully written.
1>  Done executing task "GenerateFunctionsExtensionsMetadata".
1>  Task "Move" skipped, due to false condition; ($(_IsFunctionsSdkBuild) == 'true' AND Exists('$(TargetDir)extensions.json')) was evaluated as (true == 'true' AND Exists('C:\Work\AzureFunctionsAssemblyRepro\FunctionApp1\FunctionApp1\bin\Debug\netcoreapp3.1\extensions.json')).
1>Target _FunctionsBuildCleanOutput:
1>  Using "RemoveRuntimeDependencies" task from assembly "C:\Users\steph\.nuget\packages\microsoft.azure.webjobs.script.extensionsmetadatagenerator\1.1.5\build\..\tools\net46\Microsoft.Azure.WebJobs.Script.ExtensionsMetadataGenerator.dll".
1>  Task "RemoveRuntimeDependencies"
1>  Done executing task "RemoveRuntimeDependencies".
1>Target "DotNetPublish" skipped, due to false condition; ( '$(DeployOnBuild)' == 'true' ) was evaluated as ( '' == 'true' ).
1>Target "_PackAsBuildAfterTarget" skipped, due to false condition; ('$(GeneratePackageOnBuild)' == 'true' AND '$(IsInnerBuild)' != 'true') was evaluated as ('false' == 'true' AND '' != 'true').
1>
1>Done building project "FunctionApp1.csproj".
1>
1>Build succeeded.
1>    0 Warning(s)
1>    0 Error(s)
1>
1>Time Elapsed 00:00:01.94
```

### Microsoft.NET.Sdk.Functions v3.0.3 (Microsoft.Azure.WebJobs.Script.ExtensionsMetadataGenerator v1.1.3)

If you roll back to v3.0.3, your build will succeed, the assemblies are present in the output directory, and the function succeeds at runtime. The 3.0.3 build does *not* use the `RemoveRuntimeDependencies` task:

```
1>  Using "GenerateFunctions" task from assembly "C:\Users\steph\.nuget\packages\microsoft.net.sdk.functions\3.0.3\build\..\tools\netcoreapp3.0\\Microsoft.NET.Sdk.Functions.MSBuild.dll".
1>  Task "GenerateFunctions"
1>    Function generator path: 'dotnet'
1>    Microsoft.NET.Sdk.Functions.Generator.dll "C:\Work\AzureFunctionsAssemblyRepro\FunctionApp1\FunctionApp1\bin\Debug\netcoreapp3.1\bin\FunctionApp1.dll " "C:\Work\AzureFunctionsAssemblyRepro\FunctionApp1\FunctionApp1\bin\Debug\netcoreapp3.1\ " "False "
1>  Done executing task "GenerateFunctions".
1>Target "DotNetPublish" skipped, due to false condition; ( '$(DeployOnBuild)' == 'true' ) was evaluated as ( '' == 'true' ).
1>Target _GenerateFunctionsExtensionsMetadataPostBuild:
1>  Using "GenerateFunctionsExtensionsMetadata" task from assembly "C:\Users\steph\.nuget\packages\microsoft.azure.webjobs.script.extensionsmetadatagenerator\1.1.3\build\..\tools\net46\Microsoft.Azure.WebJobs.Script.ExtensionsMetadataGenerator.dll".
1>  Task "GenerateFunctionsExtensionsMetadata"
1>    Extensions generator working directory: 'C:\Users\steph\.nuget\packages\microsoft.azure.webjobs.script.extensionsmetadatagenerator\1.1.3\tools\net46\..\netstandard2.0\generator'
1>    Extensions generator path: 'dotnet'
1>    Microsoft.Azure.WebJobs.Script.ExtensionsMetadataGenerator.Console.dll "C:\Work\AzureFunctionsAssemblyRepro\FunctionApp1\FunctionApp1\bin\Debug\netcoreapp3.1\bin" "C:\Work\AzureFunctionsAssemblyRepro\FunctionApp1\FunctionApp1\bin\Debug\netcoreapp3.1\bin\extensions.json"
1>    Found 54 assemblies to evaluate in 'C:\Work\AzureFunctionsAssemblyRepro\FunctionApp1\FunctionApp1\bin\Debug\netcoreapp3.1\bin':
1>      FunctionApp1.dll
1>        Resolving assembly: 'System.Runtime, Version=4.2.2.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a'
1>        Assembly 'System.Runtime' loaded.
1>      Microsoft.AspNetCore.Authentication.Abstractions.dll
1>      Microsoft.AspNetCore.Authentication.Core.dll
1>      Microsoft.AspNetCore.Authorization.dll
1>      Microsoft.AspNetCore.Authorization.Policy.dll
1>      Microsoft.AspNetCore.Hosting.Abstractions.dll
1>      Microsoft.AspNetCore.Hosting.Server.Abstractions.dll
1>      Microsoft.AspNetCore.Http.Abstractions.dll
1>      Microsoft.AspNetCore.Http.dll
1>      Microsoft.AspNetCore.Http.Extensions.dll
1>      Microsoft.AspNetCore.Http.Features.dll
1>      Microsoft.AspNetCore.JsonPatch.dll
1>      Microsoft.AspNetCore.Mvc.Abstractions.dll
1>      Microsoft.AspNetCore.Mvc.Core.dll
1>      Microsoft.AspNetCore.Mvc.Formatters.Json.dll
1>      Microsoft.AspNetCore.Mvc.WebApiCompatShim.dll
1>      Microsoft.AspNetCore.ResponseCaching.Abstractions.dll
1>      Microsoft.AspNetCore.Routing.Abstractions.dll
1>      Microsoft.AspNetCore.Routing.dll
1>      Microsoft.AspNetCore.WebUtilities.dll
1>      Microsoft.Azure.WebJobs.dll
1>      Microsoft.Azure.WebJobs.Host.dll
1>      Microsoft.Azure.WebJobs.Host.Storage.dll
1>      Microsoft.Build.Framework.dll
1>      Microsoft.Build.Utilities.Core.dll
1>      Microsoft.DotNet.PlatformAbstractions.dll
1>      Microsoft.Extensions.Configuration.Abstractions.dll
1>      Microsoft.Extensions.Configuration.Binder.dll
1>      Microsoft.Extensions.Configuration.dll
1>      Microsoft.Extensions.Configuration.EnvironmentVariables.dll
1>      Microsoft.Extensions.Configuration.FileExtensions.dll
1>      Microsoft.Extensions.Configuration.Json.dll
1>      Microsoft.Extensions.DependencyInjection.Abstractions.dll
1>      Microsoft.Extensions.DependencyInjection.dll
1>      Microsoft.Extensions.DependencyModel.dll
1>      Microsoft.Extensions.FileProviders.Abstractions.dll
1>      Microsoft.Extensions.FileProviders.Physical.dll
1>      Microsoft.Extensions.FileSystemGlobbing.dll
1>      Microsoft.Extensions.Hosting.Abstractions.dll
1>      Microsoft.Extensions.Hosting.dll
1>      Microsoft.Extensions.Logging.Abstractions.dll
1>      Microsoft.Extensions.Logging.Configuration.dll
1>      Microsoft.Extensions.Logging.dll
1>      Microsoft.Extensions.ObjectPool.dll
1>      Microsoft.Extensions.Options.ConfigurationExtensions.dll
1>      Microsoft.Extensions.Options.dll
1>      Microsoft.Extensions.Primitives.dll
1>      Microsoft.IdentityModel.Logging.dll
1>      Microsoft.IdentityModel.Tokens.dll
1>      Microsoft.Net.Http.Headers.dll
1>      Microsoft.WindowsAzure.Storage.dll
1>      NCrontab.Signed.dll
1>      Newtonsoft.Json.Bson.dll
1>      Newtonsoft.Json.dll
1>    'C:\Work\AzureFunctionsAssemblyRepro\FunctionApp1\FunctionApp1\bin\Debug\netcoreapp3.1\bin\extensions.json' successfully written.
1>  Done executing task "GenerateFunctionsExtensionsMetadata".
1>  Task "Move" skipped, due to false condition; ($(_IsFunctionsSdkBuild) == 'true' AND Exists('$(TargetDir)extensions.json')) was evaluated as (true == 'true' AND Exists('C:\Work\AzureFunctionsAssemblyRepro\FunctionApp1\FunctionApp1\bin\Debug\netcoreapp3.1\extensions.json')).
1>Target "_PackAsBuildAfterTarget" skipped, due to false condition; ('$(GeneratePackageOnBuild)' == 'true' AND '$(IsInnerBuild)' != 'true') was evaluated as ('false' == 'true' AND '' != 'true').
1>
1>Done building project "FunctionApp1.csproj".
1>
1>Build succeeded.
1>    0 Warning(s)
1>    0 Error(s)
1>
1>Time Elapsed 00:00:02.10
```


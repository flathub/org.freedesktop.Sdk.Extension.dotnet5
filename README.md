# Dotnet 5 SDK extension

## How to use
You need to add following lines to flatpak manifest:
```json
"sdk-extensions": [
    "org.freedesktop.Sdk.Extension.dotnet5"
],
"build-options": {
    "append-path": "/usr/lib/sdk/dotnet5/bin",
    "append-ld-library-path": "/usr/lib/sdk/dotnet5/lib",
    "env": {
        "PKG_CONFIG_PATH": "/app/lib/pkgconfig:/app/share/pkgconfig:/usr/lib/pkgconfig:/usr/share/pkgconfig:/usr/lib/sdk/dotnet5/lib/pkgconfig"
    }
},
```

###  Scripts
* `install.sh` - copies dotnet runtime to package.
* `install-sdk.sh` - copies dotnet SDK to package.

### Publishing project

```json
"build-commands": [
    "install.sh",
    "dotnet publish -c Release YourProject.csproj",
    "cp -r --remove-destination /run/build/YourProject/bin/Release/net5.0/publish/ /app/bin/",
]
```

### Using nuget packages
If you want to use nuget packages it is recommended to use the [Flatpak .NET Generator](https://github.com/flatpak/flatpak-builder-tools/tree/master/dotnet) tool. It generates sources file that can be included in manifest.

```json
"build-commands": [
    "install.sh",
    "dotnet publish -c Release --source ./nuget-sources YourProject.csproj",
    "cp -r --remove-destination /run/build/YourProject/bin/Release/net5.0/publish/ /app/bin/"
],
"sources": [
    "sources.json",
    "..."
]
```

### Publishing self contained app and trimmed binaries
Dotnet 5 gives option to include runtime in published application and trim their binaries. This allows you to significantly reduce the size of the package and get rid of `/usr/lib/sdk/dotnet5/bin/install.sh`. 

First you need to have following lines in `sources.json`. These packages are needed to build a project for specific runtime. 

```json
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm/5.0.15/microsoft.aspnetcore.app.runtime.linux-arm.5.0.15.nupkg",
    "sha512": "18dc367eb779bf00cef983c07e2502d3b81c97f95ca447f4db370d8d7dd85f60441ab0b0a9455d2021ea2da8451fbfc6852550b0e77b6f3159f9a55039893cf2",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm.5.0.15.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm64/5.0.15/microsoft.aspnetcore.app.runtime.linux-arm64.5.0.15.nupkg",
    "sha512": "3e8a7c7b87a3d824b4579d58663b29713b59197c0b7590d33ed1ffcdb2c004ad4cc5e3034082e13a4e759516c447c8926a87fd31c1756b306d53aad456673d9d",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm64.5.0.15.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-x64/5.0.15/microsoft.aspnetcore.app.runtime.linux-x64.5.0.15.nupkg",
    "sha512": "f2e82e1d98f7f6da75a94df1d025e54750d1c319252d20d331b92eb1321d4d03334a9ac82860c1920ca23b4cdc97246b140d341066bcf99bb582dca30a6ef48e",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-x64.5.0.15.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm/5.0.15/microsoft.netcore.app.runtime.linux-arm.5.0.15.nupkg",
    "sha512": "670c43aa3e1416cda511b20d201bb0ddb34514658ec97b0a02b549709752d03297cbd9c10a55805d691555dbbb5fa86aebac42f2b05dde8033ee1e2b4af89307",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm.5.0.15.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm64/5.0.15/microsoft.netcore.app.runtime.linux-arm64.5.0.15.nupkg",
    "sha512": "4570f749be4923d681ea5e520b861966c120a4c431971a0afefa518de756fa3d895d671b07efb20b980c35ea8e5cd2b05a26893238e0408cfd5a886f2586c2a0",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm64.5.0.15.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-x64/5.0.15/microsoft.netcore.app.runtime.linux-x64.5.0.15.nupkg",
    "sha512": "3acf77648a478c9752ae2d66f31001e7be244338f440ba1a91ca0bf930f4d44c74b66174ae505d4e657678a93e384b6878506c65cd48412c21d3cf0a75d40092",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-x64.5.0.15.nupkg"
},
```

Then add build options:

```json
"build-options": {
    "arch": {
        "arm": {
            "env" : {
                "RUNTIME": "linux-arm"
            }
        },
        "aarch64": {
            "env" : {
                "RUNTIME": "linux-arm64"
            }
        },
        "x86_64": {
            "env" : {
                "RUNTIME": "linux-x64"
            }
        }
    }
},
"build-commands": [
    "mkdir -p /app/bin",
    "dotnet publish -c Release --source ./nuget-sources YourProject.csproj --runtime $RUNTIME --self-contained true",
    "cp -r --remove-destination /run/build/YourProject/bin/Release/net5.0/$RUNTIME/publish/* /app/bin/",
],
```

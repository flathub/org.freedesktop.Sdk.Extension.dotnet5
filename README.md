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
Dotnet 5 gives option to include runtime in published application and trim their binaries. This allows you to significantly reduce the size of the package and get rid of `/usr/lib/sdk/dotnet5/install.sh`. 

First you need to have following lines in `sources.json`. These packages are needed to build a project for specific runtime. 

```json
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm64/5.0.3/microsoft.aspnetcore.app.runtime.linux-arm64.5.0.3.nupkg",
    "sha512": "36fb418bba43a80650a19452fd03c52bd2b5736cdd52da9bab1e2fcacff8ad547f875ab96fa0be7da4bb7981ef64e51ae82256b1a64447a0dde9ed946d2eb1e7",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm64.5.0.3.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm/5.0.3/microsoft.aspnetcore.app.runtime.linux-arm.5.0.3.nupkg",
    "sha512": "1b1ff20ee9e54f8ad3ad2480a0b97867200c476dd7ec859ed9370be350e068421202a1d781f29867875b90f1d8886d86e55f8e603bdf1e81319c313b87ade23a",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm.5.0.3.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-x64/5.0.3/microsoft.aspnetcore.app.runtime.linux-x64.5.0.3.nupkg",
    "sha512": "944acb70bb873fef3e7204766dfacf2fb20dc670762fa46e2e4c4fdeaf3fb486187cb0d4d225a503f9af8decdc44d8d089896a37d8a644d270a48a0e6140ba98",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-x64.5.0.3.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm64/5.0.3/microsoft.netcore.app.runtime.linux-arm64.5.0.3.nupkg",
    "sha512": "5ce89398e6b2388e57e79c448be680b7d4e0c0a625241c3bf6e7e8590db981ddad71bb0fbd5f80c6a5a1ab84ade623c9f85803af42eea3cafb423acde263d1ca",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm64.5.0.3.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm/5.0.3/microsoft.netcore.app.runtime.linux-arm.5.0.3.nupkg",
    "sha512": "3b1a001d117f01eb6eba3eeecc4cfc9364e961b4fbc377646aad2a5de76cbe08114f1204540bd032b174c32810a9ec4833a53ca8310e0065b9d43ea39fb4b814",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm.5.0.3.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-x64/5.0.3/microsoft.netcore.app.runtime.linux-x64.5.0.3.nupkg",
    "sha512": "4aa301976d5d6cbe626315a4ee31c88fab2c331bbd90e25246abefa54c89331ef00c645b6df174dbf772d3779f7ba33c6bde6ee1e4eeebabd0ce812745768951",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-x64.5.0.3.nupkg"
},
```

Then add build options:

```json
"build-options": {
    "arch": {
        "aarch64": {
            "env" : {
                "RUNTIME": "linux-arm64"
            }
        },
        "arm": {
            "env" : {
                "RUNTIME": "linux-arm"
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

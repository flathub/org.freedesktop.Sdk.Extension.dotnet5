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
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm64/5.0.5/microsoft.aspnetcore.app.runtime.linux-arm64.5.0.5.nupkg",
    "sha512": "51369fda14811f9b521c41666b3a651355ec9c81cf5209ce06b08f13e0053cc83728ab632139c4bb331b0e369cd758a6b74cfc14ac811c3cc88191b22f0548fe",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm64.5.0.5.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm/5.0.5/microsoft.aspnetcore.app.runtime.linux-arm.5.0.5.nupkg",
    "sha512": "d513b3d7f8d5b9e21ac795d4c8adb86e764366bc24110ca75f1a3d795cc29029bba0b88c6e6eb580e9d1fefe9ae657259c1c7bea60d153f8786bb807459e9f05",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm.5.0.5.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-x64/5.0.5/microsoft.aspnetcore.app.runtime.linux-x64.5.0.5.nupkg",
    "sha512": "aedbaafb8035871ed4348a495cbc393d094146f26554c198a25d7a86ce6a87192da95ef5b6074faef61fab9e65e087790ff69e7914777f68d9d5415c9227d5a1",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-x64.5.0.5.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm64/5.0.5/microsoft.netcore.app.runtime.linux-arm64.5.0.5.nupkg",
    "sha512": "9e4315b3880b7e873e2e1bdb2c524317c2143e3c78ca22736b610b6c972a76944fa4c16c48d4bd424d50e574a40de1d9b2c1435460e84bda8cdb7cf0873dc15d",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm64.5.0.5.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm/5.0.5/microsoft.netcore.app.runtime.linux-arm.5.0.5.nupkg",
    "sha512": "a91849a9d6ffb57006d3b5ebd9cc407f4754bcf56efa91bc7b4b7cf0849f8116d6f420b8817605ad370d18c80ec9dd207e5cbed75a1f521c3901c9195385c729",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm.5.0.5.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-x64/5.0.5/microsoft.netcore.app.runtime.linux-x64.5.0.5.nupkg",
    "sha512": "477e344aeefe045b609ee9da1ad957a4932de0765ce9a121f9d5bc6dde59c8d0287fba28d85428b16d64460c648ec349cac899e1551028a2d6e9f70d65075f3f",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-x64.5.0.5.nupkg"
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

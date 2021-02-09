# Dotnet 5 SDK extension

## How to use
Dotnet commands can be used by calling `/usr/lib/sdk/dotnet5/bin/dotnet` file. 

###  Scripts
* `/usr/lib/sdk/dotnet5/enable.sh` - enable dotnet while building package.
* `/usr/lib/sdk/dotnet5/install.sh` - copies dotnet runtime to package.
* `/usr/lib/sdk/dotnet5/install-sdk.sh` - copies dotnet SDK to package.

### Publishing project

```json
"build-commands": [
    "/usr/lib/sdk/dotnet5/enable.sh",
    "dotnet publish -c Release YourProject.csproj",
    "cp -r --remove-destination /run/build/YourProject/bin/Release/net5.0/publish/ /app/bin/",
    "/usr/lib/sdk/dotnet5/install.sh"
]
```

The `install.sh` must be called after using SDK because it replaces symlink to the dotnet binary. It doesn't matter if you are using `install-sdk.sh`.

### Using nuget packages
If you want to use nuget packages it is recommended to use the [Flatpak .NET Generator](https://github.com/flatpak/flatpak-builder-tools/tree/master/dotnet) tool. It generates sources file that can be included in manifest.

```json
"build-commands": [
    "/usr/lib/sdk/dotnet5/install.sh",
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
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm64/5.0.1/microsoft.netcore.app.runtime.linux-arm64.5.0.1.nupkg",
    "sha512": "c29ed9cc9b957ec933aad14225aba00d568225b325381eaef67ba0d4c2ad998b0292485ed52c94de6a5f5f4901e93be3a94534cb6a29798f357aa0a825f16cd6",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm64.5.0.1.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-x64/5.0.1/microsoft.netcore.app.runtime.linux-x64.5.0.1.nupkg",
    "sha512": "2692453f8c7db7422f3eb9b41f96623ea111ffd18619f1f360da6b829ba0886154b8cc9b65cb9e0c66cece6c6cc0d128179b82b126beeaef1ad9122a9f103281",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-x64.5.0.1.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm/5.0.1/microsoft.netcore.app.runtime.linux-arm.5.0.1.nupkg",
    "sha512": "c48e545b401ca9816011832b937d9f2bb13b21a8afecc1686f103db9c756613328d13844019a352c87a493fd6efdacc156911a398029efad9c8a7711a39224d8",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm.5.0.1.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm64/5.0.1/microsoft.aspnetcore.app.runtime.linux-arm64.5.0.1.nupkg",
    "sha512": "270f3ff83525a69e4e5d0e6df0ad90ba924dc8b61cf263b647fa8a8684fff92f223cb8aa020e3fe1a69b6a952ce152cdb901a87ef02b692fb401dee197ccf847",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm64.5.0.1.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-x64/5.0.1/microsoft.aspnetcore.app.runtime.linux-x64.5.0.1.nupkg",
    "sha512": "1596cc7e60435327f25a29ee5bf66e90230d7610561d6ede244d62df0fba6fd0c2d1cc818fdc59dcaaa828a392267ce106f80ec17e1ed2330ef2f58f700b3ec9",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-x64.5.0.1.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm/5.0.1/microsoft.aspnetcore.app.runtime.linux-arm.5.0.1.nupkg",
    "sha512": "a9e97d01703c169e571293982139fab6e9053bde3192f0e253376186a27bce69016fb188923942900946b51c739fb50bd8897f5f732cc9c5050844d7502ff146",
     "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm.5.0.1.nupkg"
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
    "dotnet publish -c Release --source ./nuget-sources YourProject.csproj --runtime $RUNTIME --self-contained true",
    "cp -r --remove-destination /run/build/YourProject/bin/Release/net5.0/$RUNTIME/publish/ /app/bin/",
],
```

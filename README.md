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
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm/5.0.10/microsoft.aspnetcore.app.runtime.linux-arm.5.0.10.nupkg",
    "sha512": "5c3275b20fbd7e522449dc477b6fe521e6e8ca4998e990ed168cbafce4fe42eeb80869ee890d74c346b28d3230ddb11b25145064026796c2633f0ba3af1653dd",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm.5.0.10.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm64/5.0.10/microsoft.aspnetcore.app.runtime.linux-arm64.5.0.10.nupkg",
    "sha512": "bacd5281f13d0a5b3684ca639199183a4873bf884de8e4c1ebf86176fcdb2bbd730314e989abbdab6126713dccd964e17e72366423f8e519bf4333188de099ff",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm64.5.0.10.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-x64/5.0.10/microsoft.aspnetcore.app.runtime.linux-x64.5.0.10.nupkg",
    "sha512": "2c3d3d66909a10bc7183446a3b0fcdf2ccc6d3a1b99a0884780e37b75e82fb612214bee81ddabaebf93dc9e663775202b1829f24e582e96820e033ad0dae6457",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-x64.5.0.10.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm/5.0.10/microsoft.netcore.app.runtime.linux-arm.5.0.10.nupkg",
    "sha512": "6e353aac1bde0ef33c4988cc3fba1aafe618c158ae7d1ab6535d45cfa833ffd1485d249ec465e836085b62838773d02be346c07ad5e23745721f885c4dcdb857",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm.5.0.10.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm64/5.0.10/microsoft.netcore.app.runtime.linux-arm64.5.0.10.nupkg",
    "sha512": "02f1d49490a930021f0b54f170619f7e2c1e3f7da6d9d18650595d4d46f13cdf79bddf283a2faa22a958c229a3f8bf36de75f4fd8c1acfb8665da220c286aed0",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm64.5.0.10.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-x64/5.0.10/microsoft.netcore.app.runtime.linux-x64.5.0.10.nupkg",
    "sha512": "a4b2c15cd9e5bc6baf97cb71b0de54daf72aee2bf561a69120a25bf6d762e9b81cefaa925e6aecffb8f88c01cc68c1594e79d656768c6705bdc2d273cb58ef48",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-x64.5.0.10.nupkg"
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

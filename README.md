# DotNet API Basic Project Structure

* See <Kata.md>

## Steps followed

* CLI
  * `dotnet new webapi -n CoffeeShopAPI`
  * `cd CoffeeShop.Api`
  * `dotnet add CoffeeShop.Api.csproj package Swashbuckle.AspNetCore -v 6.6.2`
* C# Dev Kit
  * Cmd + Shift + P > .NET: New Project > ASP.NET Core Web API > "CoffeeShop.Api" > Default Driectory

## Add csharpier

Add dotnet tool:
`dotnet tool install csharpier --create-manifest-if-needed`

Format all:
`dotnet csharpier .`

Update `.vscode/settings.json` with format on-save with csharpier:

```json
{
  "editor.formatOnSave": true,
  "[csharp]": {
      "editor.defaultFormatter": "csharpier.csharpier-vscode"
  }
}
```

Add csharpier `.vscode/extensions.json`

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=827846 to learn about workspace recommendations.
    "recommendations": [
      "csharpier.csharpier-vscode"
    ]
}
```



## VS Code - C# Dev Kit extension
[Introductory Videos for C# in VS Code](https://code.visualstudio.com/docs/csharp/introvideos-csharp)
[Getting Started with C# in VS Code](https://code.visualstudio.com/docs/csharp/get-started)

C# Dev Kit extension is an official MS extension.

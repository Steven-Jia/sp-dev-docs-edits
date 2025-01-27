---
title: Update SharePoint Framework packages
description: Update the npm packages that you've installed in your project and those you've installed globally.
ms.date: 10/26/2020
ms.prod: sharepoint
ms.localizationpriority: high
---
# Update SharePoint Framework packages

SharePoint client-side development tools use the [npm](https://www.npmjs.com/) package manager to manage dependencies and other required JavaScript helpers. npm is typically included as part of Node.js setup.

When you create a new client-side solution, the Yeoman generator for SharePoint fetches the latest SharePoint Framework packages required for your client-side project. As you build your project, your existing packages could be outdated because there could be new versions of one or more packages available.

Based on the [release notes](https://github.com/SharePoint/sp-dev-docs/wiki#release-notes) for a particular release or the latest package, you may decide to update your SharePoint Framework packages used in your project. SharePoint Framework packages include both the npm packages you've installed in your project, for example: [@microsoft/sp-core-library](https://www.npmjs.com/package/@microsoft/sp-core-library), and npm packages installed globally, for example: [@microsoft/generator-sharepoint](https://www.npmjs.com/package/@microsoft/generator-sharepoint).

> [!TIP]
> While it may not be required, we recommend that you update the SharePoint Framework packages often so that you can get the latest changes and fixes that have been released.

## Find outdated packages

To find the outdated packages in your client-side project, including SharePoint Framework and other packages your project depends on, run the following command in a console in the same directory as your project.

```console
npm outdated
```

The command lists the following information about the packages your project depends on. This information is looked up from the **package.json** file located in the root of your project directory and npm registry.

- Current version installed in your project
- Version requested by your project (available in **package.json**)
- Latest version available

![NPM outdated packages](../../images/npm-outdated-packages-list.png)

To identify the SharePoint Framework packages, look for the package names that start with the following npm scope and prefix:

```text
@microsoft/sp-
```

Along with the SharePoint Framework packages, you may also need to update **react** and **office-ui-fabric-react** packages. Make sure you read the [Release Notes & Roadmap](../roadmap.md) for the specific release to infer which packages require updates.

### Use "npm outdated" with a project

SharePoint Server 2016 with Feature Pack 2 supports SharePoint Framework solutions. SharePoint Server 2016 uses an older version of the SharePoint Framework than the version available in SharePoint Online.

When scaffolding new projects, the SharePoint Framework Yeoman generator prompts you to choose if your solution should be using the latest version of the SharePoint Framework and be working only with SharePoint Online, or if it should use an older version of the SharePoint Framework and work with both SharePoint Server 2016 and SharePoint Online.

When you run the **npm outdated** command in a project that targets both SharePoint Online and SharePoint Server 2016, it shows you the latest versions of the SharePoint Framework packages. These versions, however, work only with SharePoint Online. If you update your solution to use these latest packages, it no longer works with SharePoint Server 2016.

When working with SharePoint Framework solutions compatible with SharePoint hosted on-premises, you should always verify which patch level the target SharePoint farm has and which version of the SharePoint Framework it supports.

## Update packages

When updating packages to newer versions, you should always use your package manager (npm or Yarn). You shouldn't edit the **package.json** file manually. If you follow the recommended practice of using a lock file, your changes to the **package.json** file would be ignored.

Start with identifying which packages need updating and which newer version you want to use. It might not always be possible for you to use the latest version of the given package because it might be incompatible with other SharePoint Framework dependencies, such as TypeScript.

For each package that you want to update, run the following command:

```console
npm install mypackage@newversion --save
```

For example, if you were using [date-fns](https://date-fns.org/) version v2.9.0 and wanted to update to version 2.16.1, you would run:

```console
npm install date-fns@2.16.1 --save
```

Updating the package by using npm installs the specified version of the package in your project and updates the version number in the **package.json** file dependencies and the lock file used in your project.

After the packages are installed, execute the following command to clean up any old build artifacts:

```console
gulp clean
```

It's recommended to delete all old packages by deleting the **node_modules** folder and download (ie: reinstall) all dependencies with the updated **package.json**. Without this step, old versions may conflict with the newer versions the next time you build the project.

```console
npm install
```

## Update your code

Depending on breaking API changes, you may have to update your existing project code and config files. For each release, the [release notes](https://github.com/SharePoint/sp-dev-docs/wiki#release-notes) highlight any such breaking changes and the modifications required to your existing code. You must make sure you update your code with those fixes.

You can always build the project to see if you've any errors and warnings by running the command in a console in your project directory:

```console
gulp build
```

## Update Yeoman generator

If you've installed the SharePoint Framework Yeoman generator globally, you can find out if it requires updating by running the following command:

```console
npm outdated -g
```

The command lists the following information about the packages installed globally on your machine. This information is looked up from the versions installed on your machine and the npm registry.

- Current version installed globally on your machine
- Version requested by you when you installed
- Latest version available

![NPM outdated global packages](../../images/npm-outdated-global-packages-list.png)

To identify the generator package, look for the following package name:

```console
@microsoft/generator-sharepoint
```

### Update generator package

Open your favorite console and execute the following command to update the generator to its latest published version:

```console
npm install @microsoft/generator-sharepoint@latest -g
```

The command updates the Yeoman generator for SharePoint to the latest published version along with its dependencies. You can validate this by executing the following command in the console:

```console
npm ls @microsoft/generator-sharepoint -g --depth=0
```

## See also

- [Upgrading SharePoint Framework projects (Team-based development guidance)](../team-based-development-on-sharepoint-framework.md#upgrading-sharepoint-framework-projects)
- [Upgrade SharePoint items (Provisioning SharePoint assets)](provision-sharepoint-assets.md#upgrade-sharepoint-items)
- [SharePoint Framework Overview](../sharepoint-framework-overview.md)

---
title: Add an external library to your SharePoint client-side web part
description: Bundle a JavaScript external library and share libraries.
ms.date: 01/25/2021
ms.prod: sharepoint
ms.localizationpriority: high
---

# Add an external library to your SharePoint client-side web part

You might want to include one or more JavaScript libraries in your web part. This article shows you how to bundle an external library and share libraries.

## Bundle a script

By default, the web part bundler automatically includes any library that is a dependency of the web part module. This means that the library is deployed in the same JavaScript bundle file as your web part. This is more useful for smaller libraries that aren't used in multiple web parts.

### Example

1. Include the string validating library [validator](https://www.npmjs.com/package/validator) package into a web part.
1. Download the validator package from NPM:

    ```console
    npm install validator --save
    ```

    > [!NOTE]
    > Because you're using TypeScript, you'll likely want the type declarations for the package you add. This is useful when you are writing code because TypeScript is just a superset of JavaScript. All the TypeScript code is still converted to JavaScript code when you compile. You can install type declarations by using **NPM**, for example: **npm install @types/{package} --save-dev**

1. Create a file in your web part's folder called `validator.d.ts` and add the following:

    ```typescript
    declare module "validator" {
        export function isEmail(email: string): boolean;
        export function isAscii(text: string): boolean;
    }
    ```

    > [!NOTE]
    > Some libraries do not have type declarations. While the validator library does have a [community provided type declarations file](https://www.npmjs.com/package/@types/validator), for this scenario let's assume it does not. In this case, you would want to define your own type declarations definition `.d.ts` file for the library. The previous code shows an example.

1. In your web part file, import the type declarations:

    ```typescript
    import * as validator from 'validator';
    ```

1. Use the validator library in your web part code:

    ```typescript
    validator.isEmail('someone@example.com');
    ```

## Share a library among multiple web parts

Your client-side solution might include multiple web parts. These web parts might need to import or share the same library. In such cases, instead of bundling the library, you should include it in a separate JavaScript file to improve performance and caching. This is especially true of larger libraries.

### Example

In this example, you'll share the [marked](https://www.npmjs.com/package/marked) package (a markdown compiler) in a separate bundle.

1. Download the **marked** package from NPM:

    ```console
    npm install marked --save
    ```

1. Install the type declarations package in your project:

    ```console
    npm install @types/marked --save-dev
    ```

1. Edit the **config/config.json**, and add an entry to the `externals` map. This is what tells the bundler to put this in a separate file. This prevents the bundler from bundling this library:

    ```json
    "marked": "node_modules/marked/marked.min.js"
    ```

1. Add the statement to import the **marked** library in your web part now that we've added the package and type declarations for the library:

    ```typescript
    import * as marked from 'marked';
    ```

1. Use the library in your web part:

    ```typescript
    console.log(marked('I am using __markdown__.'));
    ```

## Load a script from a CDN

Instead of loading the library from an NPM package, you might want to load a script from a Content Delivery Network (CDN). To do so, edit the **config.json** file to load the library from its CDN URL.

### Example

In this example, you'll load jQuery from CDN. You don't need to install the NPM package. However, you still need to install the type declarations.

1. Install the type declarations for jQuery:

    ```console
    npm install --save-dev @types/jquery
    ```

1. Update the **config.json** in the **config** folder to load jQuery from CDN. Add an entry to the `externals` field:

    ```json
    "jquery": "https://code.jquery.com/jquery-3.1.0.min.js"
    ```

1. Import jQuery in your web part:

    ```typescript
    import * as $ from 'jquery';
    ```

1. Use jQuery in your web part:

    ```javascript
    alert( $('#foo').val() );
    ```

## Load a non-AMD module

Some scripts follow the legacy JavaScript pattern of storing the library on the global namespace. This pattern is now deprecated in favor of [Universal Module Definitions (UMD)](https://github.com/umdjs/umd)/[Asynchronous Module Definitions (AMD)](https://en.wikipedia.org/wiki/Asynchronous_module_definition) or [ES6 modules](https://github.com/lukehoban/es6features/blob/master/README.md#modules). However, you might need to load such libraries in your web part.

> [!TIP]
> It's difficult to determine manually whether the script that you're trying to load is an AMD or a non-AMD script. This is especially the case if the script that you're trying to load is minified.
>
> If your script is hosted on a publicly accessible URL, you can use the free [Rencore SharePoint Framework Script Check](https://rencore.com/script-check/) tool to determine the type of script for you. Additionally, this tool lets you know whether the hosting location from which you're loading the script is properly configured. This tool is also available in the VS Code extension [Rencore SPFx Script Check](https://marketplace.visualstudio.com/items?itemName=RencoreGmbH.vscode-spfx-script-check)

To load a non-AMD module, you add an additional property to the entry in your **config.json** file.

### Example

In this example, you'll load a fictional non-AMD module from Contoso's CDN. These steps  apply for any non-AMD script in your src/ or node_modules/ directory.

The script is called **Contoso.js** and is stored at **https://contoso.com/contoso.js**. Its contents are:

```javascript
var ContosoJS = {
  say: function(text) { alert(text); },
  sayHello: function(name) { alert('Hello, ' + name + '!'); }
};
```

1. Create type declarations for the script in a file called **contoso.d.ts** in the web part folder.

    ```typescript
    declare module "contoso" {
      interface IContoso {
        say(text: string): void;
        sayHello(name: string): void;
      }
      var contoso: IContoso;
      export = contoso;
    }
    ```

1. Update the **config.json** file to include this script. Add an entry to the **externals** map:

    ```json
    {
      "contoso": {
        "path": "https://contoso.com/contoso.js",
        "globalName": "ContosoJS"
      }
    }
    ```

1. Add an import to your web part code:

    ```typescript
    import contoso from 'contoso';
    ```

1. Use the contoso library in your code:

    ```typescript
    contoso.sayHello(username);
    ```

## Load a library that has a dependency on another library

Many libraries have dependencies on another library. You can specify such dependencies in the **config.json** file by using the `globalDependencies` property.

> [!IMPORTANT]
> Note that you don't have to specify this field for AMD modules; they properly import each other. However, a non-AMD module cannot have an AMD module as a dependency. In some cases, it is possible to load an AMD script as a non-AMD script. This is often required when working with jQuery, which by itself is an AMD script, and jQuery plug-ins, which most of the time are distributed as non-AMD scripts and which depend on jQuery.

There are two examples of this.

### Non-AMD module has a non-AMD module dependency

This example involves two fictional scripts. These are in the **src/** folder, although they can also be loaded from a CDN.

**ContosoUI.js**

```javascript
Contoso.EventList = {
  alert: function() {
    var events = Contoso.getEvents();
    events.forEach( function(event) {
      alert(event);
    });
  }
}
```

**ContosoCore.js**

```javascript
var Contoso = {
  getEvents: function() {
    return ['A', 'B', 'C'];
  }
};
```

1. Add or create type declarations for this class. In this case, you'll create **Contoso.d.ts**, which contains type declarations for both JavaScript files.

    **contoso.d.ts**

    ```typescript
    declare module "contoso" {
      interface IEventList {
        alert(): void;
      }
      interface IContoso {
        getEvents(): string[];
        EventList: IEventList;
      }
      var contoso: IContoso;
      export = contoso;
    }
    ```

1. Update the **config.json** file. Add two entries to `externals`:

    ```json
    {
      "contoso": {
        "path": "/src/ContosoCore.js",
        "globalName": "Contoso"
      },
      "contoso-ui": {
        "path": "/src/ContosoUI.js",
        "globalName": "Contoso",
        "globalDependencies": ["contoso"]
      }
    }
    ```

1. Add imports for Contoso and ContosoUI:

    ```typescript
    import contoso = require('contoso');
    require('contoso-ui');
    ```

1. Use the libraries in your code:

    ```typescript
    contoso.EventList.alert();
    ```

## Load SharePoint JSOM

Loading SharePoint JSOM is essentially the same scenario as loading non-AMD scripts that have dependencies. This means using both the `globalName` and `globalDependency` options.

> [!IMPORTANT]
> Note that the following approach causes errors in classic SharePoint pages, where SharePoint JSOM is already loaded. If you require your web part to work with both classic and modern pages, you should first check if SharePoint JSOM is already available, and if it isn't, load it dynamically by using the **SPComponentLoader**.

1. Install type declarations for Microsoft Ajax, which is a dependency for JSOM type declarations:

    ```console
    npm install @types/microsoft-ajax --save-dev
    ```

1. Install type declarations for the JSOM:

    ```console
    npm install @types/sharepoint --save-dev
    ```

1. Add entries to the `config.json`:

    ```json
    {
      "sp-init": {
        "path": "https://CONTOSO.sharepoint.com/_layouts/15/init.js",
        "globalName": "$_global_init"
      },
      "microsoft-ajax": {
        "path": "https://CONTOSO.sharepoint.com/_layouts/15/MicrosoftAjax.js",
        "globalName": "Sys",
        "globalDependencies": [ "sp-init" ]
      },
      "sp-runtime": {
        "path": "https://CONTOSO.sharepoint.com/_layouts/15/SP.Runtime.js",
        "globalName": "SP",
        "globalDependencies": [ "microsoft-ajax" ]
      },
      "sharepoint": {
        "path": "https://CONTOSO.sharepoint.com/_layouts/15/SP.js",
        "globalName": "SP",
        "globalDependencies": [ "sp-runtime" ]
      }
    }
    ```

1. In your web part, add the following `require` statements:

    ```typescript
    require('sp-init');
    require('microsoft-ajax');
    require('sp-runtime');
    require('sharepoint');
    ```

## Load localized resources

You can use a map in **config.json** called `localizedResources` to describe how to load localized resources. Paths in this map are relative to the **lib** folder and must not contain a leading slash (**/**).

In this example, you have a folder **src/strings/**. In this folder, there are several JavaScript files with names such as **en-us.js**, **fr-fr.js**, **de-de.js**. Because each of these files must be loadable by the module loader, they must contain a CommonJS wrapper. For example, in **en-us.js**:

```typescript
  define([], function() {
    return {
      "PropertyPaneDescription": "Description",
      "BasicGroupName": "Group Name",
      "DescriptionFieldLabel": "Description Field"
    }
  });
```

1. Edit the **config.json** file. Add an entry to `localizedResources`. `{locale}` is a placeholder token for the locale name:

    ```json
    {
      "strings": "strings/{locale}.js"
    }
    ```

1. Add type declarations for your strings. In this case, you have a file **MyStrings.d.ts**:

    ```typescript
    declare interface IStrings {
      webpartTitle: string;
      initialPrompt: string;
      exitPrompt: string;
    }

    declare module 'mystrings' {
      const strings: IStrings;
      export = strings;
    }
    ```

1. Add imports for the strings in your project:

    ```typescript
    import * as strings from 'mystrings';
    ```

1. Use the strings in your project:

    ```typescript
    alert(strings.initialPrompt);
    ```

## See also

- [SharePoint Framework Overview](../../sharepoint-framework-overview.md)

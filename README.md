
## twinkle-starter

This is a template repository to create a Twinkle customisation for a new wiki. 

To use this template, click on the ![image](https://user-images.githubusercontent.com/6702424/98155461-92395e80-1ed6-11eb-93b2-98c64453043f.png) button near the top. An automated workflow will shortly create a new commit in your repository replacing the placeholders in the package.json file.

## Customising Twinkle - Getting Started
You need to have the following installed on your system: (i) Git, (ii) Node.js v13 or above, (iii) npm.

After cloning the repo generated by this template, run `npm install`. If `npm install` doesn't work and you're using npm v7, try `npm install --legacy-peer-deps`. 

This repo contains has all the dependencies and build tool configurations present so that you don't have to bother with them.

Twinkle-core uses [TypeScript](https://en.wikipedia.org/wiki/TypeScript) - a compiled superset of JavaScript that makes it easier to write bug-free applications. You can choose to write your customisations in either JavaScript or TypeScript. However, using JavaScript sidesteps all the type-safety features built into twinkle-core, which may make it harder to debug. If you are new to TypeScript, refer to the [official documentation's introduction for JS programmers](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html).

There exists automatically generated **[Code Documentation for twinkle-core](https://tools-static.wmflabs.org/twinkle/core-docs/modules.html)**.

### File structure

### twinkle.ts 
`twinkle.ts` file is the entry point. Specify all site-specific configurations here. The list of module classes is also put here.

### Extending core modules
As of now, twinkle-core provides the following modules: 
- [Fluff](https://twinkle.toolforge.org/core-docs/classes/fluffcore.html)
- [Diff](https://twinkle.toolforge.org/core-docs/classes/diffcore.html)  
- [Tag](https://twinkle.toolforge.org/core-docs/classes/tagcore.html) - Add maintenance tags to pages 
- [XFD](https://twinkle.toolforge.org/core-docs/classes/xfdcore.html) - Nominate pages for deletion discussions.
- [Speedy](https://twinkle.toolforge.org/core-docs/classes/speedycore.html) - Nominate pages for speedy deletion, and (for admins) delete pages
- [Warn](https://twinkle.toolforge.org/core-docs/classes/warncore.html) - Warn users for vandalism or other issues
- [Block](https://twinkle.toolforge.org/core-docs/classes/blockcore.html) - (for admins) Block users and place a block template on the talk page
- [Protect](https://twinkle.toolforge.org/core-docs/classes/protectcore.html) - Request page protection, (for admins) protect pages and add protection templates 
- [Unlink](https://twinkle.toolforge.org/core-docs/classes/unlinkcore.html) - Remove links to a given page from all other pages
- [BatchDelete](https://twinkle.toolforge.org/core-docs/classes/batchdeletecore.html) - (for admins) batch page deletions
- [BatchUndelete](https://twinkle.toolforge.org/core-docs/classes/batchundeletecore.html) - (for admins) batch undeletion of pages linked from a given page

Refer to the documentation of each module for guidance on configuring them. Fluff, Diff, Unlink, BatchDelete and BatchUndelete require virtually no configuration.

Each module in Twinkle extends the abstract class [TwinkleModule](https://twinkle.toolforge.org/core-docs/classes/twinklemodule.html). Some core module classes are also abstract, indicating they don't work by themselves and must be extended with the specified abstract fields or methods that you have to provide.  

Customisation written for English Wikipedia at [wikimedia-gadgets/twinkle-enwiki](https://github.com/wikimedia-gadgets/twinkle-enwiki) can be used as a reference.
 

### Creating your own modules
Apart from extending the core module, you can also create your own modules customised to the needs of your wiki. In general, a Twinkle module is structured as follows:

```js
// A singleton class: only one instance is ever created
class MyCustomModule extends TwinkleModule {
	
	moduleName = 'CustomModule';
	
	// Entry point. Calling the constructor should create the menu item for this module
	constructor() {
		// addMenu() call creates the menu item in the Twinkle portlet
		addMenu();
    }
	
	// When the menu item is clicked, this function is invoked,
    // Creates the dialog with a form into which a user can enter inputs  
	makeWindow() {  }
	
	// Triggerred when the form is submitted 
	evaluate() {  }
}
```


## Development

Commands:
- `npm start` - this creates a quick build of the project which you can test by loading `mw.loader.load('http://localhost:5500/twinkle.js');` from your on-wiki common.js page (or from the browser console). 
- `grunt build` - this creates a minified single-file build that you copy over to the wiki (see Deployment below).

## Writing automated tests
Twinkle-starter comes with a test suite which uses [Jest](https://www.npmjs.com/package/jest), [mock-mediawiki](https://www.npmjs.com/package/mock-mediawiki) and [playwright](https://www.npmjs.com/package/playwright). Unit tests for utility functions can be written with mock-mediawiki for mocking any `mw.*` functions if necessary. 

For writing integration tests, you can spin up a MediaWiki instance using Docker (`npm run test:integration:setup`), use playwright for browser automation, and mwn for setting up test fixtures via the API and checking results. However, note that writing integration tests is quite time-consuming and is likely overkill, unless you plan to make frequent changes.

## Deployment

To generate a production build, run `grunt build`. This build minimises the code and surrounds it within nowiki tags after escaping all existing nowiki tags. A header is also inserted. The morebits.js and morebits.css files are copied over from twinkle-core dependency.

#### Deploy as a gadget
Edit MediaWiki:Gadgets-definition to add the Twinkle gadget:
```
*Twinkle[ResourceLoader|dependencies=ext.gadget.morebits,ext.gadget.select2,mediawiki.api|type=general|peers=Twinkle-pagestyles]|Twinkle.js|Twinkle.css
*morebits[ResourceLoader|dependencies=mediawiki.user,mediawiki.util,mediawiki.Title,jquery.ui|hidden]|morebits.js|morebits.css
*Twinkle-pagestyles[hidden|skins=vector]|Twinkle-pagestyles.css
*select2[ResourceLoader|hidden]|select2.min.js|select2.min.css
```

Copy the [MediaWiki:Gadget-select2.min.js](https://en.wikipedia.org/wiki/MediaWiki:Gadget-select2.min.js) and [MediaWiki:Gadget-select2.min.css](https://en.wikipedia.org/w/index.php?title=MediaWiki:Gadget-select2.min.css) files as-it-is from enwiki. These are _not_ exact copies of the CDN versions. A small change is applied to them to make them work with MediaWiki ResourceLoader. These files should never need to be edited once created.

You can deploy manually by copying all files in the build directory (created by running `grunt build`) to the wiki.

You can also use the [deploy.js script](https://github.com/wikimedia-gadgets/twinkle-starter/blob/master/deploy.js) to push to the wiki. 
- Create a bot password or set up OAuth credentials. Ensure you provide the sufficient rights.
- Optional: Create a `credentials.json` file with your login information. If this is not done, the deploy script will prompt you for the username and password.
- Adjust the `deployTargets` field in deploy.js script as appropriate. Run it.

#### Deploy as a user script

Twinkle can also be made to work as a userscript. However, this will be slightly slower than the gadget version.

After running `grunt build`, copy the files from the build directory to your userspace on-wiki. Skip Twinkle-pagestyles.css - it is not useful in a userscript context.

Copy [MediaWiki:Gadget-select2.min.js](https://en.wikipedia.org/wiki/MediaWiki:Gadget-select2.min.js) and [MediaWiki:Gadget-select2.min.css](https://en.wikipedia.org/w/index.php?title=MediaWiki:Gadget-select2.min.css) files as-it-is from enwiki to your userspace. 

Create a central loader (say [[User:Example/twinkle.js]]):
```js
mw.loader.using(['mediawiki.user', 'mediawiki.util', 'mediawiki.Title', 'mediawiki.api']).then(function () {
	function load(pageName, css) {
		return mw.loader.getScript(
			'/w/index.php?title=User:Example' + pageName + '&action=raw&ctype=text/' + css ? 'css' : 'javascript', 
            css ? 'text/css' : null  
        );
    }
    mw.loader.load('jquery.ui');
	
    // Morebits needs to be available for twinkle.js to parse
	load('twinkle/morebits.js').then(function() {
		load('twinkle/twinkle.js');
    });
	load('twinkle/select2.js');
	load('twinkle/twinkle.css', true);
	load('twinkle/morebits.css', true);
	load('twinkle/select2.css', true);
});
```

In theory, the fetch of i18n messages and twinkleconfig could be parallelized with fetching of the twinkle source files to make loads faster ([Issue 3](https://github.com/wikimedia-gadgets/twinkle-core/issues/3)). 

## Development customisation

Twinkle-start is a rather opinionated toolkit so that you don't have to configure any development tooling, but you always tweak them. Examples: 

### Don't use TypeScript
<details>
    <summary>Click to expand</summary>
Rename all `.ts` files to `.js`. 

Remove any type specifiers and other non-JS syntax you see anywhere.

Run `npm uninstall typescript ts-loader ts-jest @typescript-eslint/eslint-plugin @typescript-eslint/parser`

Run `npm i -D babel-loader`

Modify `webpack.config.js` and `webpack.prod.config.json` to use <a href="https://www.npmjs.com/package/babel-loader">babel-loader</a> instead of ts-loader.

Modify `eslintrc.json` file to remove mentions of typescript parser and plugin.
</details>

### Remove prettier

<details>
    <summary>Click to expand</summary>
[Prettier](https://prettier.io) is opinionated, it is OK to want to break free from it.

Run the command:
```bash
npm uninstall prettier lint-staged husky
```
Remove the "husky" and "lint-staged" fields in package.json.

</details>

### Use a source code host other than GitHub
<details>
  <summary>Click to expand</summary>
Temporarily create a GitHub repository for the template initialisation workflow. When it completes, clone the repo, after which you can delete the GitHub repo and push it to the source code hosting site of your preference.

Delete the `.github` directory - everything inside it works only on GitHub. You'll have to find your own means for any CI workflows you may want to run.
</details>

----

Developed as part of [Grants:Project/Rapid/SD0001/Twinkle localisation](https://meta.wikimedia.org/wiki/Grants:Project/Rapid/SD0001/Twinkle_localisation).

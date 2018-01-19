# Manage Libs Versions

Easily create a list of Radio Inputs to let user choose Library Versions,
retrieve the selection and prepare a list of Assets to be dynamically loaded.

The Assets list can be directly passed to
[`loadJsCss.list()`](https://github.com/ghybs/load-js-css) function.

[![GitHub releases](https://img.shields.io/github/release/ghybs/manage-libs-versions.svg?label=GitHub)](https://github.com/ghybs/manage-libs-versions/releases)
[![npm](https://img.shields.io/npm/v/manage-libs-versions.svg)](https://www.npmjs.com/package/manage-libs-versions)


## Quick usage guide

This library uses a UMD wrapper.

If directly loaded into a browser, it assigns the `manageLibsVersions` global variable.


## Example

**HTML**
```html
<script src="manage-libs-versions.js"></script>
```

**JavaScript**
```javascript
var lib1PathTemplateCss = 'https://unpkg.com/lib1@{{VERSION}}/dist/lib1.css';
var lib1PathTemplateJS = 'https://unpkg.com/lib1@{{VERSION}}/dist/lib1.js';

var lib2PathTemplateJs = 'https://unpkg.com/lib2@{{VERSION}}/dist/lib2.js';

var bundle = new manageLibsVersions.Bundle({
  name: 'bundle',
  libs: [{
    name: 'lib1',
    mandatory: true,
    versions: [{
      name: 'v2.0.0',
      defaultVersion: true, // If no Version is specified, or the specified one is unknown, use this Version instead.
      assets: [
          manageLibsVersions.makeStylesheet(lib1PathTemplateCss, '2.0.0'),
          manageLibsVersions.makeScript(lib1PathTemplateJS, '2.0.0')
      ]
    }, {
     name: 'v1.0.0',
     defaultVersion: false,
     disabled: true, // This will make the Radio Input disabled.
     assets: [
         manageLibsVersions.makeStylesheet(lib1PathTemplateCss, '1.0.0'),
         manageLibsVersions.makeScript(lib1PathTemplateJS, '1.0.0')
     ]
   }]
  }, {
    name: 'lib2',
    versions: [{
      name: 'dev',
      assets: [{
        type: 'script',
        path: '../dist/bundle.js'
      }]
    }, {
      name: 'v0.15.2',
      assets: [
          manageLibsVersions.makeScript(lib2PathTemplateJs, '0.15.2')
      ]
    }]
  }]
});
```

**HTML**
```html
<p>Choose a version for lib1:</p>
<!-- Versions names can be separated by whitespace, comma (,) and/or semi-colon (;) -->
<div data-manage-lib="lib1" data-manage-versions="v2.0.0, v1.0.0"></div>

<p>Choose a version for lib2:</p>
<div data-manage-lib="lib2" data-manage-versions="dev, v0.15.2"></div>

<button id="reload">Reload with the above selected Versions</button>

<script src="https://unpkg.com/urijs@1.18.12/src/URI.min.js"></script>
<script src="https://unpkg.com/load-js-css@0.0.1/index.js"></script>
```

**JavaScript**
```javascript
// Fill manage placeholders with Radio Inputs.
bundle.fillPlaceholders();

// Make sure lib2 dev version assets are available.
// If not, it will automatically disable the Radio Input and remove
// the defaultVersion flag.
bundle.getLibVersion('lib2', 'dev').checkAssetsAvailability(true)
    .then(function () {
      loadDependenciesAndPageScript();
    })
    .catch(function () {
      bundle.getLibVersion('lib2', 'v0.15.2').defaultVersion = true;
      loadDependenciesAndPageScript();
    });

function loadDependenciesAndPageScript() {
  // Read in the URL which Libraries Versions are requested.
  var url = window.location.href;
  // https://github.com/medialize/URI.js
  var urlParts = URI.parse(url);
  var queryStringParts = URI.parseQuery(urlParts.query);
  
  // Get the Libraries Versions assets list and select the Radio Inputs,
  // so that the user sees which Versions will be loaded.
  var list = bundle.getAndSelectVersionsAssetsList(queryStringParts);
  
  // Finally include the page script, now that lib1 and lib2 dependencies
  // are available.
  list.push({
    type: 'script',
    path: './index.js'
  });
  
  // https://github.com/ghybs/load-js-css
  loadJsCss.list(list, {
    // Load scripts after stylesheets, delayed by this duration (in ms).
    // Typically if a Library expects that its stylesheet is applied
    // when its script executes.
    delayScripts: 500
  });
}


document.getElementById('reload').addEventListener('click', function (event) {
  event.preventDefault();

  // Read the checked Radio Inputs (i.e. which Versions the user has selected).
  var bundleVersions = bundle.readSelectedVersionsNames();
  // Build a new URL including those Versions requests as Query part.
  var url = new URI(window.location.href).setQuery(bundleVersions);

  // Reload the page with the new requested Versions.
  window.location.href = url.toString();
});
```


## License

Manage Libs Versions library is distributed under the [ISC License](https://choosealicense.com/licenses/isc/).

---
layout: post
title: Using Ember Visual Acceptance
author: Eric White
description: "Using ember-cli-visual-acceptance with ember-frost-file-picker"
modified: 2016-07-13
tags: [ember-cli-visual-acceptance]
---

# Installation
 Simple as running `ember install ember-cli-visual-acceptance`

## Testem.json

In order for `ember-cli-visual-acceptance` to capture images, `"launch_in_ci"` must contain ethier `SlimerJsVisualAcceptance` or `PhantomJsVisualAcceptance`. 
 
Example Testem.json:

~~~ javascript
{
  "framework": "mocha",
  "test_page": "tests/index.html?hidepassed",
  "disable_watching": true,
  "launch_in_ci": [
    "SlimerJsVisualAcceptance"
  ],
  "launch_in_dev": [
    "Firefox"
  ],
  "launchers": {
    "PhantomJsVisualAcceptance": {
      "command": "phantomjs vendor/phantomjs-launcher.js <url>",
      "protocol": "browser"
    },
    "SlimerJsVisualAcceptance": {
      "command": "slimerjs -jsconsole vendor/phantomjs-launcher.js <url>",
      "protocol": "browser"
    }
  }
}
~~~

# Usage
Inside an integration or acceptance test you can use `return capture('<image-label>')` at the end of an `it()` function.

~~~ javascript
it('renders', function () {
  // Set any properties with this.set('myProperty', 'value')
  // Handle any actions with this.on('myAction', function (val) { ... })

  this.render(hbs`{{frost-file-picker}}`)
  expect(this.$('.frost-file-picker')).to.have.length(1)
  return capture('Basic-File-Picker')
})
~~~

 You can also use this functionality for async testing:

~~~ javascript
it('renders', function (done) {
  // Set any properties with this.set('myProperty', 'value')
  // Handle any actions with this.on('myAction', function (val) { ... })

  this.render(hbs`{{frost-file-picker}}`)
  expect(this.$('.frost-file-picker')).to.have.length(1)
  capture('Basic-File-Picker').then(function (data) {
    console.log(arguments)
    done()
  }).catch(function (err) {
    done(err)
  })
})
~~~

In this case done comes from `it('test', function (done) { ... }`. You can also use the `then` functionality to get creative rather than using done.
`arguments` will display the response from ResembleJS or the message, `'No passed image. Saving current test as base'` (If there is no current baseline image).

## Usage with Qunit
To use with Qunit you must supply the `assert` parameter.

Example (Using [ember-cli-showdown](https://github.com/gcollazo/ember-cli-showdown/blob/master/tests/integration/components/markdown-to-html-test.js)): 

~~~ javascript
import hbs from 'htmlbars-inline-precompile';
import { moduleForComponent, test } from 'ember-qunit';

moduleForComponent('markdown-to-html', 'Integration | Component | markdown to html', {
  integration: true
});

test('positional parameter', function(assert) {
  assert.expect(1);

  this.set('markdown', '*hello world*');
  this.render(hbs`{{markdown-to-html markdown}}`);
  assert.equal(this.$('> *').html(), '<p><em>hello world</em></p>');
  return capture('Markdown', null, null, 0.00, assert);
});
~~~

### Larger frame
In order to go beyond the default 640x340px dimensions you can supply the width and height to the capture function. The following shows an example to get a 1920x4041 container: `capture('<image-label>', 1920, 4041)`.

# Setting up travis
Replace `ember test` with `ember tva`. This command comes with `ember-cli-visual-acceptance` and provides the functionality for commenting  a report (Stored on Imgur) on a PR from the Travis build.


Add your github credentials to `before_script:`

~~~ yaml
before_script:
- git config --global user.email "ericwhite613@gmail.com"
- git config --global user.name "Eric White"
- git config --global push.default simple
- npm set progress=false
- git config credential.helper "store --file=.git/credentials"
- echo "https://${GH_TOKEN}:@github.com" > .git/credentials
~~~

**Secure Variables (Display value in build log = `OFF`)**

* Add `GH_TOKEN` to Travis. Personal Access Token must have push access

**Unsecure Variables (Display value in build log = `ON`)**

* Add `RO_GH_TOKEN` Unsecure token that can only read.

* Add `VISUAL_ACCEPTANCE_TOKEN` token, value can be found [here](https://travis-ci.org/ciena-frost/ember-frost-file-picker/jobs/137522760#L275)
  * If you put the `VISUAL_ACCEPTANCE_TOKEN` directly in your code and commit it to Github; Github will revoke the token.

## Browsers - html2canvas vs. PhantomJS render callback

You must enable the display to use headless browsers by adding the following to the `before_script` hook: 

~~~ yaml
before_script:
- "export DISPLAY=:99.0"
- "sh -e /etc/init.d/xvfb start"
- sleep 3 # give xvfb some time to start
~~~

### PhantomJS - SlimerJS

[PhantomJS](http://phantomjs.org/) and [SlimerJS](https://slimerjs.org/) can both be used with this tool to capture images.

Personally I prefer using SlimerJS as their version of Gecko matches the latest Firefox. While PhantomJS Webkit is about a year behind Safari's Webkit version. `SlimerJsVisualAcceptance` images come out much more accurate. Additionally, debugging the images produced from the `.ember-testing-container` in Firefox is useful. Since the `.ember-testing-container` is identical in SlimerJS and Firefox ( at least I've never seen a difference between the two).

#### Warning

With certain repositories I've had trouble with SlimerJS having segmentation faults on both Linux and Mac. I've yet to resolve this issue.

### Html2Canvas

Html2Canvas is used when a browser does not have the function `window.callPhantom` (Only PhantomJS and SlimerJS have this defined). Html2Canvas is still in beta and as result you will see some issues.
Html2Canvas relies on Canvas drawing support. I find Chrome has the best Canvas drawing support (miles ahead of their competitors), while Firefox has the second best Canvas drawing support. 

#### SVGs

Html2Canvas has difficulties rendering SVGs (more so in Firefox than in Chrome). As a result I have added a new **expermental** functionality that attempts to render the svgs better.
You can use this experimental feature by setting `experimentalSvgs` to `true` (Example: `capture('svg-experimental', null, null, null, true)`).

Experimental SVGs will not be used for PhantomJS and SlimerJS as their rendering handles SVGs (since it's basically just a simple screenshot of the page)

### Using Firefox
To use Firefox in Travis simply set

~~~ javascript
  // Testem.json
  "launch_in_ci": [
    "Firefox"
  ],
~~~
And add the following to your `.travis.yml` to get the latest version of Firefox:
~~~ yaml 
addons:
  firefox: "latest"
~~~

### Using SlimerJS
Testem.json

~~~ javascript
{
  "framework": "mocha",
  "test_page": "tests/index.html?hidepassed",
  "disable_watching": true,
  "launch_in_ci": [
    "SlimerJsVisualAcceptance"
  ],
  "launch_in_dev": [
    "Firefox"
  ],
  "launchers": {
    "PhantomJsVisualAcceptance": {
      "command": "phantomjs vendor/phantomjs-launcher.js <url>",
      "protocol": "browser"
    },
    "SlimerJsVisualAcceptance": {
      "command": "slimerjs -jsconsole vendor/phantomjs-launcher.js <url>",
      "protocol": "browser"
    }
  }
}
~~~

### Using PhantomJS
Testem.json

~~~ javascript
{
  "framework": "mocha",
  "test_page": "tests/index.html?hidepassed",
  "disable_watching": true,
  "launch_in_ci": [
    "PhantomJsVisualAcceptance"
  ],
  "launch_in_dev": [
    "Safari"
  ],
  "launchers": {
    "PhantomJsVisualAcceptance": {
      "command": "phantomjs vendor/phantomjs-launcher.js <url>",
      "protocol": "browser"
    },
    "SlimerJsVisualAcceptance": {
      "command": "slimerjs -jsconsole vendor/phantomjs-launcher.js <url>",
      "protocol": "browser"
    }
  }
}
~~~

## Reports for  multiple browsers

Producing a report for multiple browsers is perfectly fine. All you need to do is add your collection of browsers to `launch_in_ci`.

Example:

~~~ javascript
// Testem.json
"launch_in_ci": [
    "Firefox",
    "SlimerJsVisualAcceptance"
  ],
~~~

### Notes
* Travis will upload the reports to Imgur

If you would like to help or have ideas on improving this tool I'm available on the Ember community slack @ewhite613 - issues and PRs also welcome :) 

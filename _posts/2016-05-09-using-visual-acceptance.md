---
layout: post
title: Using Ember Visual Acceptance
author: Eric White
description: "Using ember-cli-visual-acceptance with ember-frost-file-picker"
modified: 2016-05-29
tags: [ember-cli-visual-acceptance]
---

# Installation
 Simple as running `ember install ember-cli-visual-acceptance`

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
 Html2Canvas does not handle the `zoom` css property that Mocha uses to scale the `ember-testing` container to `50%`. In order to view a larger container if you wish to go beyond the default 640x340px dimensions you can supply the width and height to the capture function. The following shows an example to get a 1920x4041 container: `capture('<image-label>', 1920, 4041)`.

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

## Browsers
I prefer using non-headless browsers as their Canvas drawing support (used by [html2canvas](http://html2canvas.hertzen.com/)) is usually better. I find Chrome has the best drawing support, and Firefox second. SlimerJS's Canvas support is probably the best for being a headless browser.

### Using latest firefox
Add this to your .travis.yml

~~~ yaml
addons:
  firefox: "latest"
~~~

And enable the display by adding the following to the `before_script` hook: 

~~~ yaml
before_script:
- "export DISPLAY=:99.0"
- "sh -e /etc/init.d/xvfb start"
- sleep 3 # give xvfb some time to start
~~~

### Using PhantomJS
You can also use PhantomJS. We have included bluebird (A library that provides promises) and JQuery in your vendor files. These libraries are both imported when running `ember test`. I recommend using PhantomJS 1.9.8 rather than 2.0. I've had weird experiences with 2.0 but you are welcome to try it.

### SlimerJS
Slimerjs is also another good option. But I've had trouble trying to get SlimerJS launcher to close on Linux with Mocha.
Here is the launcher I used for `testem.js`


~~~ javascript
/* Testem usage
'launchers': {
    'slimerjs': {
      'command': 'slimerjs slimerjs-launcher.js <url>',
      'protocol': 'browser'
    }
  }
*/
// Doesn't exit on Linux for some reason
'use strict'
var system = require('system')
var page = require('webpage').create()
var url = system.args[1]
page.viewportSize = {
  width: 1024,
  height: 768
}

page.open(url)
page.onError = function (msg, trace) {
  console.log(msg)
  trace.forEach(function (item) {
    console.log('  ', item.file, ':', item.line)
  })
}
~~~

### Notes
* Travis will upload the reports to Imgur

If you would like to help or have ideas on improving this tool I'm available on the Ember community slack @ewhite613 - issues and PRs also welcome :) 

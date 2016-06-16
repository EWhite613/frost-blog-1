---
layout: post
title: Using Ember Visual Acceptance
author: Eric White
description: "Using ember-cli-visual-acceptance with ember-frost-file-picker"
modified: 2016-05-09
tags: [ember-cli-visual-acceptance]
---

# Installation
 Simple as running `ember install ember-cli-visual-acceptance`
## Requirements
Must be using Mocha

# Usage
Inside an integration or acceptance test you can use `return capture('<image-label>')` at the end of an `it()` function. You can also use this functionality if you wish to:

~~~ javascript
capture('<image-label>', <width>, <height>).then(function (data) {
  console.log(arguments)
  done()
}).catch(function (err) {
  done(err)
})
~~~

In this case done comes from `it('test', function (done) { ... }`. But you can also use this format to get creative rather than using done.
`arguments` will display the response from ResembleJS or the message, 'No passed image. Saving current test as base', if it matches the scenario described.
### Larger frame
 Html2Canvas does not handle the `zoom` css property that Mocha uses to scale the `ember-testing` container to `50%`. In order to view a larger container if you wish to go beyond the default 640x340px dimensions you can supply the width and height to the capture function. The following shows an example to get 1920x1080 container: `capture('<image-label>', 1920, 1080)`
# Setting up travis
Replace `ember test` with `ember tva --pr-api-url=http://openshiftvisualacceptance-ewhite.rhcloud.com/comment`

Add your github credentials to `before_script:`

~~~ yaml
- git config --global user.email "<github-email>"
- git config --global user.name "<user-name>"
- git config --global push.default simple
- npm set progress=false
- git config credential.helper "store --file=.git/credentials"
- echo "https://${GH_TOKEN}:@github.com" > .git/credentials
~~~
**Secure Variables (Display value in build log = `OFF`)**

* Add GH_TOKEN to Travis. Personal Access Token must have push access

**Unsecure Variables (Display value in build log = `ON`)**

* Add RO_GH_TOKEN Unsecure token that can only read.

* Add VISUAL_ACCEPTANCE_TOKEN token, value can be found [here](https://travis-ci.org/ciena-frost/ember-frost-file-picker/jobs/137522760#L275)

## Using latest firefox
Add this to your .travis.yml

~~~ yaml
addons:
  firefox: "latest"
~~~
And enable the display

~~~ yaml
before_script:
- "export DISPLAY=:99.0"
- "sh -e /etc/init.d/xvfb start"
- sleep 3 # give xvfb some time to start
~~~

## Using PhantomJS
You can also use PhantomJS. We have included bluebird (A library that provides promises) and JQuery in your vendor files. These libraries are both imported when running `ember test`. I recommend using PhantomJS 1.9.8 rather than 2.0. I've had weird experiences with 2.0 but you are welcome to try it.

## SlimerJS
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
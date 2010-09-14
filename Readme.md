
# Soda

 Selenium Node Adapter.

## Features

  - Saucelabs integration
  - Optional chaining API to prevent callback nesting

## Actions

"Selenese" actions include commands such as _open_ and _type_. Every action has a corresponding `Client` method which accept a variable number of arguments followed by a callback `Function` which receives any potential `err`, the response `body`, and `response` object itself. 

    browser.session(function(err){
      browser.open('/', function(err, body, res){
        browser.type('q', 'Hello World', function(err, body, res){
          browser.testComplete(function(){
            
          });
        });
      });
    });

Because nested callbacks can quickly become overwhelming, Soda has optional chaining support by simply utilizing the `.chain` getter as shown below. If an exception is thrown in a callback, or a command fails then it will be passed to `end(err)`.

    browser
      .chain
      .session()
      .open('/')
      .type('q', 'Hello World')
      .testComplete()
      .end(function(err){
        if (err) throw err;
        console.log('done');
      });

When chaining successful commands may receive a callback, which is useful for custom assertions:

    browser
      .chain
      .session()
      .open('/')
      .getTitle(function(title){
        assert.equal('Hello World', title);
      })
      .testComplete()
      .end(function(err){
        if (err) throw err;
      })

## Saucelab Videos &amp; Logs

When a job is complete, you can request the log or flv video from saucelabs. To access the url for these resources you may use `SauceClient#videoUrl` or `SauceClient#logUrl`, for example:

    ...
    .end(function(err){
      console.log(this.videoUrl)
      console.log(this.logUrl)
    })

Saucelabs also provides a script that you may embed in your CI server to display the video, accessible via `SauceClient#video`, which will yield something similar to:

    <script src="http://saucelabs.com/video-embed/<job-id>.js?username=<username>&access_key=<access-key>"/>

## Selenium RC Example

    var soda = require('../index')
      , assert = require('assert');

    var browser = soda.createClient({
        host: 'localhost'
      , port: 4444
      , url: 'http://www.google.com'
      , browser: 'firefox'
    });

    browser
      .chain
      .session()
      .open('/')
      .type('q', 'Hello World')
      .clickAndWait('btnG')
      .getTitle(function(title){
        assert.ok(~title.indexOf('Hello World'))
      })
      .testComplete()
      .end(function(err){
        if (err) throw err;
        console.log('done');
      });


## Saucelabs Example

    var soda = require('./index')
      , assert = require('assert');

    var browser = soda.createSauceClient({
        'url': 'http://sirrobertborden.ca.app.learnboost.com/'
      , 'username': '<your username>'
      , 'access-key': '<your api key>'
      , 'os': 'Linux'
      , 'browser': 'firefox'
      , 'browser-version': '3.'
      , 'max-duration': 300 // 5 minutes
    });

    // Log commands as they are fired
    browser.on('command', function(cmd, args){
      console.log(' \x1b[33m%s\x1b[0m: %s', cmd, args.join(', '));
    });

    browser
      .chain
      .session()
      .setTimeout(8000)
      .open('/')
      .waitForPageToLoad(5000)
      .clickAndWait('//input[@value="Submit"]')
      .clickAndWait('link=Settings')
      .type('user[name][first]', 'TJ')
      .clickAndWait('//input[@value="Save"]')
      .assertTextPresent('Account info updated')
      .clickAndWait('link=Log out')
      .testComplete()
      .end(function(err){
        if (err) throw err;
        console.log('done');
      });  

## Creating Helpers

Keep in mind you can extend the prototype as needed for your test. An example of this which we frequently use is `waitForDialog()`. Since the exports of `require('soda')` is the `Client` itself we can extend it as shown below, in our case waiting for an element with the class of ".dialog" to be present.

    soda.prototype.waitForDialog = function() {
      return this.waitForElementPresent('css=.dialog');
    };

## Running The Test Suite

 First we need to start Selenium RC:
 
     $ java -jar selenium-server.jar

 Then run:
 
     $ make test

## More Information

  - Saucelabs  [Supported Browsers](http://saucelabs.com/products/docs/sauce-ondemand/browsers)
  - Introduction to [Selenese](http://seleniumhq.org/docs/02_selenium_basics.html)
  - Selenium [Command Reference](http://release.seleniumhq.org/selenium-core/0.8.0/reference.html).


## License 

(The MIT License)

Copyright (c) 2010 LearnBoost &lt;dev@learnboost.com&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

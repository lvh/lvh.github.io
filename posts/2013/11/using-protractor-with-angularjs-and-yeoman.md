<!--
.. title: Using protractor with AngularJS and Yeoman
.. date: 2013/11/15 11:29
.. slug: using-protractor-with-angularjs-and-yeoman
.. link:
.. description:
.. tags: 
-->

In writing the website for Crypto 101, my upcoming book, I've been
toying around with AngularJS. The basic setup was done using Yeoman.

First of all, the Yeoman Angular generator gives you a
`karma-e2e.conf.js`, but doesn't really configure grunt to do any
end-to-end testing. Turns out that the new hotness for Angular
scenario testing is based on Protractor. Karma isn't deprecated, it's
just for unit tests. Protractor is based on Selenium, which is
probably an upgrade :-)

Anyway, here's what I had to do to get started.

1. In the root of your app, install protractor and the Selenium
   standalone server.

```
npm install --save-dev protractor
./node_modules/protractor/bin/install_selenium_standalone
```

2. Optionally, add `selenium` to your `.gitignore`, since it contains
   a ~30MB binary that doesn't belong in your git repo.

3. Add a `protractor.conf.js` file. Here's mine. Remember that there's
   some stuff in there for you to uncomment, so don't just copy paste.
   On my machine, the default driver was Internet Explorer for
   Windows, which is strange given that I'm running on OS X. Oh well.
   This uses Chrome instead.

```javascript
exports.config = {
  seleniumAddress: 'http://localhost:4444/wd/hub',
  specs: [
    // To run plain JS files, uncomment the following line:
    // './test/integration/*.js',
    // To run Coffeescript files, uncomment the following line:
    // '.tmp/integration/*.js'
  ],

  // Set capabilities.
  capabilities: {
    'browserName': 'chrome'
  },

  jasmineNodeOpts: {
    showColors: true,
    defaultTimeoutInterval: 30000
  }
}
```

4. If you're using Coffeescript, change your Gruntfile to build your
   integration tests:

```
  grunt.initConfig({
    // ...
    coffee: {
      // ...
      test: {
        files: [
        // ...
        {
          expand: true,
          cwd: 'test/integration',
          src: '{,*/}*.coffee',
          dest: '.tmp/integration',
          ext: '.js'
        }]
      }
    },
  // ...
})
```

5. Write some tests in `test/integration`.

6. You can now manually run your tests. First run Selenium:

```
./node_modules/protractor/bin/install_selenium_standalone
```

Then run your tests:

```
./node_modules/protractor/bin/protractor protractor.conf.js
```

Once I figure out the proper way to do this in Grunt, I'll let you
know.

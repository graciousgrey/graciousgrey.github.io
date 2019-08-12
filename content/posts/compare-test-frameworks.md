+++
title= "Comparison of Test Frameworks - TestCafe, Nightwatch.js and Cypress"
date= 2019-06-25T17:05:22+02:00
+++
For our <a href="https://github.com/photoprism/photoprism">open source project</a> we have been looking for a test framework to execute end to end tests. 
Photoprism is a photo-management tool written in go and using vuetify for the frontend.

Our main demands on an acceptance test framework are:
     
* It must be open source
* Tests must be stable
* Tests must run from within a docker container (linux)
* Tests must be run by travis.ci
* Tests must be easy to read/write
* Tests must be run in several browsers
* Not too many dependencies must be needed
 
We decided to test the following frameworks: <a href="https://devexpress.github.io/testcafe/">TestCafe</a>, <a href="https://nightwatchjs.org/">Nightswatch.js</a> and <a href="https://www.cypress.io/">Cypress</a>.
Whereas Nightwatch uses the selenium webdriver, TestCafe and Cypress are selenium independent.

### Browser Support:


##### Nightwatch:
Using nightwatch you can cover the main browsers (see table). Therefore you need to install the appropriate webdrivers and configure them.

|  |  |
| --------|----------|
| Browser Support | Geckodriver, Chromedriver, Safaridriver, Microsoft Webdriver |
| Browser Support Headless | All mentioned above |

##### TestCafe:
TestCafe convinces with a lot of officially supported browsers (see table). Besides those you can try to run your tests in all other browsers you have installed on your machine.
I ran the tests without problems on Samsung IE as well. And no configuration is needed.

|  |  |
| --------|----------|
| Browser Support | Chrome, Chromium, Chrome-Canary, Firefox, Safari, Opera, Edge, IE |
| Browser Support Headless | Chrome, Chromium, Firefox |

##### Cypress:
Cypress only supports a few browsers. No configuration is needed.

|  |  |
| --------|----------|
| Browser Support | Chrome, Chromium, Chrome-Canary, Electron |
| Browser Support Headless | Electron |

### Set up all frameworks locally on Mac
All frameworks can easily be installed via npm
```javascript
npm install --save-dev nightwatch
```

```javascript
npm install --save-dev testcafe
```

```javascript
npm install --save-dev cypress 
```
    
### Write test
For all framworks I show an example test, that verifies that the user can use the navigation to switch from page to page


##### Nightwatch:
In general the syntax is self-explaining and well documented.<br>
What costs me some time was to get the tests stable.
The most common issue here: I want to click an element but it is not interactable. 
Some of these issues can be solved by playing with waitForElementVisible(Selector:enabled) and waitForElementNotVisible, but not all.<br>
To use Selectors like  Selector('a').withText(option) you need to install an extension (<a href="https://github.com/css2xpath/css2xpath">css2xpath</a>).
<br>Testing xhr requests is also not included out of the box (<a href="https://github.com/cortexmg/nightwatch-xhr">nightwatch-xhr</a>)


 ```javascript
module.exports = {
    'Test navigation' : function (browser) {
        let photos = browser.page.photosPage();
        browser
            .url('http://localhost:2342/photos')
        photos.openNav();
        browser.click('a[href="/albums"]')
            .assert.visible('div.p-page-albums');
        photos.openNav();
        browser.click('a[href="/places"]')
            .assert.visible('div.leaflet-map-pane')
        photos.openNav();
        browser.click('a[href="/labels"]')
            .assert.visible('div.p-page-labels')
        photos.openNav();
        browser.click('a[href="/library"]')
            .assert.visible('div.p-tab-upload')
            .end();
    },
};


        openNav: function() {
        this.api.waitForElementVisible('div.p-page', 15000);
        this.api.waitForElementNotVisible('#p-busy-overlay', 15000);
        this.api.element("css selector", "button.p-navigation-show", function(result){
        if (result.value && result.value.ELEMENT) {
        return this
        .click("button.p-navigation-show")
        .waitForElementVisible('a[href="/albums"]:enabled', 15000);
        }
        });
        this.api.element("css selector", "div.p-navigation-expand i", function(result){
        if (result.value && result.value.ELEMENT) {
        return this
        .click("div.p-navigation-expand i")
        .waitForElementVisible('a[href="/albums"]:enabled', 15000);
        }
        });
            }
 ```
##### TestCafe
Syntax is easy to understand and very well documented.<br>
The tests have been stable directly probably due to the <a href="https://devexpress.github.io/testcafe/documentation/test-api/built-in-waiting-mechanisms.html"> built in waiting mechanism</a>.<br>
It provides <a href="https://devexpress.github.io/testcafe/documentation/test-api/selecting-page-elements/selectors/functional-style-selectors.html">nice selector options </a> and <a href="https://devexpress.github.io/testcafe/documentation/test-api/intercepting-http-requests/" >xhr features</a> out of the box.

```javascript
fixture`Use navigation`
    .page`${testcafeconfig.url}`;

const page = new Page();

test('Navigate', async t => {
    await page.openNav();
    await t
        .click('a[href="/albums"]')
        .expect(Selector('div.p-page-albums').exists, {timeout: 5000}).ok();
    await page.openNav();
    await t
        .click('a[href="/places"]')
        .expect(Selector('div.leaflet-map-pane').exists).ok();
    await page.openNav();
    await t
        .click('a[href="/labels"]')
        .expect(Selector('div.p-page-labels').exists, {timeout: 5000}).ok();
    await page.openNav();
    await t
        .click('a[href="/library"]')
        .expect(Selector('div.p-tab-upload').exists, {timeout: 5000}).ok();
});


       async openNav() {
        if (await Selector('button.p-navigation-show').visible) {
            await t.click(Selector('button.p-navigation-show'));
        } else if (await Selector('div.p-navigation-expand').exists) {
            await t.click(Selector('div.p-navigation-expand i'));
        }
    }
```
##### Cypress:
The syntax was easy to learn and it was easy to write stable tests. 
Nice selectors and <a href="https://docs.cypress.io/guides/guides/network-requests.html#Waiting">xhr features</a> are provided by the framework.

```javascript
describe('Navigate', function() {
    it('Use navigation', function() {
        cy.server()
        cy.route('/api/v1/photos*').as('getPhotos')
        cy.route('/api/v1/albums*').as('getAlbums')
        cy.route('/api/v1/labels*').as('getLabels')

        cy.visit('http://localhost:2342/photos')
        cy.wait(['@getPhotos']).then((xhr) => {
            assert.isNotNull(xhr.response.body.data, 'Photos loaded')
        })
        cy.get('button.p-navigation-show', { timeout: 15000 }).click()
        cy.get('a[href="/albums"]', { timeout: 10000 }).click()
        cy.wait(['@getAlbums']).then((xhr) => {
           assert.isNotNull(xhr.response.body.data, 'Albums loaded')
         })
        cy.get('div.p-page-albums').should('exist')
        cy.get('button.p-navigation-show', { timeout: 10000 }).click()
        //cy.wait(5000)
        cy.get('a[href="/places"]', { timeout: 10000 }).click()
        cy.wait(['@getPhotos']).then((xhr) => {
            assert.isNotNull(xhr.response.body.data, 'Places loaded')
        })
        cy.get('div.leaflet-map-pane').should('exist')
        cy.get('button.p-navigation-show', { timeout: 15000 }).click()
        cy.get('a[href="/labels"]', { timeout: 10000 }).click()

        cy.wait(['@getLabels']).then((xhr) => {
            assert.isNotNull(xhr.response.body.data, 'Labels loaded')
        })
        cy.get('div.p-page-labels', { timeout: 15000 })
        cy.get('button.p-navigation-show', { timeout: 15000 }).click()
        cy.get('a[href="/library"]', { timeout: 10000 }).click()

        cy.get('div.p-tab-upload', { timeout: 15000 })
    })
})
})
```
### Run test locally
##### Nightwatch:
For nightwatch to run, you  first need to install a webdriver e.g chromedriver: 
```javascript
brew cask install chromedriver
```
Then you need to add the path to the config 
```javascript
"server_path": "/usr/local/bin/chromedriver"
```
Now you can execute the test:
```javascript
$(npm bin)/nightwatch nightwatch_2e2tests/navigate.js
```

##### TestCafe:
You can execute tests with:
 ```javascript
testcafe chrome frontend/tests/acceptance/navigate.js
 ```
I ran this test on my mac without problems within chrome, firefox, safari and opera.

##### Cypress:
To open the test runner use:
  ```javascript
$(npm bin)/cypress open
  ```
  Within the testrunner you can select a browser (chrome or electron) and select the test to run.

OR to run tests without the testrunner use: 
  ```javascript
$(npm bin)/cypress run --browser chrome --spec "cypress/integration/tests/navigation.spec.js"
  ```

### Run tests in headless mode
##### Nightwatch:
It took me some time to figure out how to get tests running in headless mode.
For chromedriver the following configuration worked for me:
  ```javascript
"chromeOptions": {
        "args" : ["--no-sandbox",
        "--no-cache",
        "--disable-translate",
        "--disable-extensions",
        "--disable-web-security",
        "--disable-dev-shm-usage",
        "--disable-gpu",
        "--ignore-certificate-errors",
        "--headless"
        ],
        "binary": "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
    }
  ```
You can execute tests in headless mode with:
  ```javascript
$(npm bin)/nightwatch nightwatch_2e2tests/navigate.js
  ```
##### TestCafe:
You can directly run tests in headless mode with
  ```javascript
testcafe firefox:headless tests/acceptance/navigate.js
  ```
  OR
   ```javascript
testcafe chrome:headless tests/acceptance/navigate.js
   ```
##### Cypress:
You can run tests in headless electron with:
   ```javascript
$(npm bin)/cypress run --spec "cypress/integration/tests/navigation.spec.js"
   ```
### Run tests in container
##### Nightwatch:
We installed the chromedriver and nightwatch in the container via the dockerfile.<br>
You can also install it as project dependency via npm but we want to use as few dependencies as possible.


Define the path to chromedriver in the config:
   ```javascript
"server_path": "/usr/lib/node_modules/chromedriver/bin/chromedriver"
   ```
Install <a href="https://askubuntu.com/questions/510056/how-to-install-google-chrome">chrome browser within the container</a>
and add the right path to binary:
   ```javascript
"server_path": ""binary": "/usr/bin/google-chrome-stable""
   ```
Now you can execute tests from within the container:
   ```javascript
$(npm bin)/nightwatch nightwatch_2e2tests/navigate.js
   ```
##### TestCafe:</h4>
Install testcafe, chromium and firefox via the dockerfile.<br>
Note: For the tests to run we needed to use a larger /dev/shm.<br>
Now tests can be run from within the container via:
  ```javascript
testcafe chromium:headless --disable-dev-shm-usage tests/acceptance/navigate.js
   ```
   
   ```javascript
testcafe firefox:headless tests/acceptance/navigate.js
   ```

Note: The larger dev/shm is not possible with the latest docker version that is supported by travis CI.<br>
Until a newer docker version is supported tests can only be run in chromium on travis.

<!--
#####Cypress
Install cypress within the container
Install dependencies within container
    ```javascript
apt-get install xvfb libgtk2.0-0 libnotify-dev libgconf-2-4 libnss3 libxss1 libasound2
    ```
Install browser within the container (electron) → Error: EACCES: permission denied, mkdir '/usr/lib/node_modules/electron/.electron'
(Cypress:4193): Gtk-WARNING **: 13:39:10.684: cannot open display: :99</p>
-->
### Duration of test execution 

The table shows the execution time of the example test in chrome (TestCafe and Nightwatch) and electron (Cypress)

| Nightwatch | |
|------|------------|
|Headed |9-16s |
|Headless |9-10s |

| TestCafe | |
|------|----------|
|Headed |15-18s |
|Headless |10-13s |

| Cypress | |
|------|---------|
|Headed |12-14s|
|Headless |14-15s|

In headless mode (which will be used for CI) nightwatch.js performed better than TestCafe and Cypress.

### Conclusion

##### Nightwatch

| Pros |
|------|
| + Tests are fastest |

| Cons |
|----------|
| - Time consuming to write stable tests|
| - Selectors are not  as nice as in cypress/testcafe - dependecy needed (<a href="https://github.com/css2xpath/css2xpath">css2xpath</a>) |
| - Testing xhr requests is only possible with extension (e.g. <a href="https://github.com/cortexmg/nightwatch-xhr">nightwatch-xhr</a>) |
| - In addition to the framework itself and the browser webdrivers need to be installed and configured|

##### TestCafe

| Pros |
|------|
| + Super easy to write stable tests|
| + Lots of browsers are supported without the need for separate configuration|
| + No dependencies needed - only testcafe and browsers need to be installed|
| + Nice <a href="https://devexpress.github.io/testcafe/documentation/test-api/intercepting-http-requests/" >xhr features</a> out of the box|
| + It is possible to run tests locally on remote mobile devices <a href="https://devexpress.github.io/testcafe/documentation/recipes/test-on-remote-computers-and-mobile-devices.html">via QR code|

|Cons |
|----------|
| - Tests are slower|

##### Cypress

| Pros |
|------|
| + Easy to write stable tests |
| + Nice <a href="https://docs.cypress.io/guides/guides/network-requests.html#Waiting">xhr features</a> out of the box |
| + Nice <a href="https://docs.cypress.io/guides/core-concepts/test-runner.html#Command-Log">test runner</a>, especially helpful for debugging|

|Cons |
|----------|
| - Only a few browsers are supported |
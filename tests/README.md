# About
This is a simple todo app using react


## Available Scripts
In the project directory, you can run:

### `yarn start`
Runs the app in the development mode.
Open http://localhost:3000 to view it in the browser.

The page will reload if you make edits.
You will also see any lint errors in the console.

### `yarn build`

Builds the app for production to the `build` folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.



# Plan:

- Clone repository, install cucumber and playwright
 
- creating feature using gherkin https://cucumber.io/docs/gherkin/reference/
	- use table (join tests)

- creating step definition (playwright)
	- locators https://playwright.dev/docs/locators#creating-locators How choose: https://www.w3schools.com/cssref/css_selectors.asp
	- action https://playwright.dev/docs/api/class-page
	- check api response after action https://playwright.dev/docs/api/class-page#page-wait-for-response
	- assertion https://playwright.dev/docs/test-assertions

- run test in different browsers https://playwright.dev/docs/test-configuration#multiple-browsers

- try to fix failed test
	- find problem. use tracing https://playwright.dev/docs/trace-viewer
	- debug https://playwright.dev/docs/debug#pagepause

- Configuration
	- slow_mo
	- try emulation https://playwright.dev/docs/test-configuration#emulation




# Precondition: node.js must be installed

# Steps:

1. **Set up** 

- set up playwright
	- `npm i -D @playwright/test`
	- `npm i -D playwright` 
	- `npx playwright install`

- set up cucumber
	- `npm i -D @cucumber/cucumber@7.3.1 @cucumber/pretty-formatter`

- after that check your package.json: 
		"@cucumber/cucumber": "^7.3.1",
	    "@cucumber/pretty-formatter": "^1.0.0",
	    "@playwright/test": "^1.25.1",
	    "playwright": "^1.25.1"


2. **Create config file conf.js**

const { Before, BeforeAll, AfterAll, After, setDefaultTimeout } = require("@cucumber/cucumber");
const { chromium } = require("playwright"); 

setDefaultTimeout(60000) // cucumber limitation

// launch the browser
BeforeAll(async function () {
   global.browser = await chromium.launch({
       headless: false,
   });

});

// close the browser
AfterAll(async function () {
   await global.browser.close();
});

// Create a new browser context and page per scenario
Before(async function () {
   global.context = await global.browser.newContext();
   global.page = await global.context.newPage();
});

// Cleanup after each scenario
After(async function () {
   await global.page.close();
   await global.context.close();
});



3. **create feature file**: tests/feature/logIn.feature 
using gherkin. 


4. create context file for inplementation of the test tests/stepDefinition/context.js

5. **create run command in package.json** 
	"scripts": {
    	"test:e2e": "cucumber-js --require conf.js --require tests/stepDefinition/*.js --format @cucumber/pretty-formatter"
  	},


6. **try to run test in terminal**:
	yarn test:e2e tests/feature/logIn.feature

	in output should have:

	1) Scenario: log in # tests/feature/firstTest.feature:5
   	✔ Before # conf.js:23
   	? When user1 logs in with username "Marie" and password "radioactivity"
       Undefined. Implement with the following snippet:

         When('user1 logs in with username {string} and password {string}', function (string, string2) {
           // Write code here that turns the phrase above into concrete actions
           return 'pending';
         });


7. **creating step definition (playwright)**
	copy paste to tests/stepDefinition/context.js (function should be async function)
		When('user logs in with username {string} and password {string}', function (string, string2) {
           // Write code here that turns the phrase above into concrete actions
           return 'pending';
         });

8. **require cucumber and playwright in context.js**:
	const {When} = require('@cucumber/cucumber')
	const {expect} = require('@playwright/test')
	
9. **add const url = 'https://localhost:9200'**
	and add first step: await page.goto(url)

10. **run test again**:

	if you have error: page.goto: net::ERR_CERT_AUTHORITY_INVALID at https://localhost:9200/
		add ignore to conf.js: global.context = await global.browser.newContext({ ignoreHTTPSErrors: true });

	expected: test passed

11. **try to use test generator**:
	in console: npx playwright codegen playwright.dev
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

- set up cucumber
	- `npm i -D @cucumber/cucumber@7.3.1 @cucumber/pretty-formatter`

- after that check your package.json: 
		
	`"@cucumber/cucumber": "^7.3.1",`
	`"@cucumber/pretty-formatter": "^1.0.0",`
    	`"@playwright/test": "^1.25.1",`
    	`"playwright": "^1.25.1"``


2. **Create config file conf.js**
```
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
```


3. **create feature file**: tests/feature/login.feature 
	create a login test using the gherkin syntax

4. **create empty context file ** for inplementation of the test tests/stepDefinition/context.js

5. **create run command in package.json** 
	add to "scripts"- section
    	`"test:e2e": "cucumber-js --require conf.js --require tests/stepDefinition/*.js --format @cucumber/pretty-formatter"`
  	
6. **try to run command in terminal**:
	`yarn test:e2e tests/feature/login.feature`

	as result we should have in terminal:
	```
	1) Scenario: log in # tests/feature/login.feature:5
   	✔ Before # conf.js:23
   	? When user with username "Marie" and password "radioactivity" logs in
       Undefined. Implement with the following snippet:

         When('user with username {string} and password {string} logs in', function (string, string2) {
           // Write code here that turns the phrase above into concrete actions
           return 'pending';
         });
	```

7. **creating step definition (playwright)**
	copy paste to tests/stepDefinition/context.js (function should be async function)
		```When('user logs in with username {string} and password {string}', async function (string, string2) {
           	// Write code here that turns the phrase above into concrete actions
           	return 'pending';
         	});```

8. **require cucumber and playwright in context.js**:
	`const {When} = require('@cucumber/cucumber')`
	`const {expect} = require('@playwright/test')`
	
	and add const url. Use https://localhost:9200 if you have running ocis localy
 	`const url = 'https://localhost:9200'`

9. ** add first step** to test body
	`wait page.goto(url)`

10. **run test again**:

	if you have error: `page.goto: net::ERR_CERT_AUTHORITY_INVALID at https://localhost:9200/`
		add ignore to conf.js: `global.context = await global.browser.newContext({ ignoreHTTPSErrors: true });`

	expected: test passed

11. **try to use test generator**:
	in console: `npx playwright codegen playwright.dev`

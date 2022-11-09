# Presentation:

- Introduction to web testing: https://docs.google.com/presentation/d/1P12z0zLeARcCBn_d_A67rOlcAjlzW8LIHIK6LQtlMKs 

- Clone repository, install cucumber and playwright 
	- `git clone https://github.com/ScharfViktor/e2e-workshop.git`
 

# What we are going to do:

- set up playwright and cucumber
- write test using gherkin syntax
- implement step definition using playwright
- using test generator
- checks the api response in the e2e test for more confidence
- run test in different browsers
- configuration
- debug test
- tracing



## Precondition: node.js must be installed

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
    	`"@playwright/test": "^1.27.1",`
    	`"playwright": "^1.27.1"`


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
	see how do it: https://cucumber.io/docs/gherkin/reference/

4. **create empty context file** for inplementation of the test tests/stepDefinition/context.js

5. **create run command in package.json** 
	add to "scripts"- section
    	`"test:e2e": "cucumber-js --require conf.js --require tests/stepDefinition/*.js --format @cucumber/pretty-formatter"`
  	
6. **try to run command in terminal**:
	`pnpm test:e2e tests/feature/login.feature`

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
	
	and add const url. Use https://localhost:9200 if you have running ocis localy or use test instance
 	`const url = 'https://ocis.ocis-traefik.released.owncloud.works'`

9. **add first step** to test body
	`await page.goto(url)`

10. **run test again**:

	if you have error: `page.goto: net::ERR_CERT_AUTHORITY_INVALID at https://localhost:9200/`
		add ignore to conf.js: `global.context = await global.browser.newContext({ ignoreHTTPSErrors: true });`

	expected: test passed

11. **try to use test generator**:
	in console: `npx playwright codegen playwright.dev`
	you can copy paste generated code from `Playwright Inspector` to contex.js or create a test yourself 
	use: 
		- locators https://playwright.dev/docs/locators#creating-locators How choose: https://www.w3schools.com/cssref/css_selectors.asp
		- action https://playwright.dev/docs/api/class-page
		- assertion https://playwright.dev/docs/test-assertions

12. **create new test**: user creates folder and checks api response after creating folder
	-  check api response after action https://playwright.dev/docs/api/class-page#page-wait-for-response

13. **try to run test in different browsers** 
	instead of `chromium` in `conf.js` use `webkit` or `firefox`. Playwrigth launches safari even if you use linux or windows

14. **configuration** 

15. **debug and tracing**
	debug: https://playwright.dev/docs/debug#pagepause
	tracing: https://playwright.dev/docs/trace-viewer
	
	- Set the trace path dir in conf: 
	```
	global.browser = await chromium.launch({
        headless: false,
        tracesDir: 'tests/trace' 
   	});
	```
	- in section `Before` in conf after `browser.newContext()`, enable tracing start
	`await context.tracing.start({ screenshots: true, snapshots: true, sources: true})`
	- in section `After` in conf before `await global.page.close()`, disable tracing
	`await context.tracing.stop({ path: 'tests/trace/trace.zip' });`
	- run test
	- open https://trace.playwright.dev/ and select `zip` file from `tests/trace/trace.zip`

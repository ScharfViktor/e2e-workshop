# Presentation:

- Introduction to web testing: https://cloud.owncloud.com/index.php/s/BX8KvgsfDVKv6K0 

- Clone repository, install cucumber and playwright 
	- `git clone https://github.com/ScharfViktor/e2e-workshop.git`
 

# What we are going to do:

- today: introduction to end-to-end (e2e) testing with playwright
  - set up playwright and cucumber
  - write test using gherkin syntax
  - implement step definition using playwright
  - using test generator
  - check api responses in the e2e test for more confidence
  - run test in different browsers
  - configuration
  - debug tests
  - tracing
- next time: get familiar with e2e tests in github.com/owncloud/web

## Precondition: node.js must be installed

# Steps:

## 1. Setup
- set up playwright
	- `npm i -D @playwright/test playwright`

- set up cucumber
	- `npm i -D @cucumber/cucumber@7.3.1 @cucumber/pretty-formatter`

- after that check your package.json, it should contain:
	```json
    "devDependencies": {
      "@cucumber/cucumber": "^7.3.1",
      "@cucumber/pretty-formatter": "^1.0.0",
      "@playwright/test": "^1.27.1",
      "playwright": "^1.27.1"
    }
    ```

## 2. Create config file `conf.js`
Copy & paste following content into the file `conf.js`
```js
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

## 3. Create script in package.json
Copy & paste the following string into the `scripts` section of your `package.json`:

```json
"scripts": {
  "test:e2e": "cucumber-js --require conf.js --require tests/stepDefinition/*.js --format @cucumber/pretty-formatter"
}
```

You can run all defined scripts with `npm` by their keys from within the context of your project later on.

## 4. Create feature file
- Create an empty file `tests/feature/login.feature`
- Gherkin reference: https://cucumber.io/docs/gherkin/reference/
- Copy & paste the following content
	```gherkin
    Feature: Login
      Logging in and stuff...

      Scenario: log in
        To fill out: try to write a step which logs in user "marie" with password "radioactivity"
    ```
- Try to fill out the missing line in Gherkin :-)

## 5. Create context file for step definitions
- Create an empty file `tests/stepDefinition/context.js`
- Copy and paste the following content
	```js
    const {When} = require('@cucumber/cucumber')
    const {expect} = require('@playwright/test')
 
    const url = 'https://ocis.ocis-traefik.released.owncloud.works'
	```

## 6. Run test with the `test:e2e` script
- run command `npm run test:e2e tests/feature/login.feature` in your terminal
- result should be similar to:
	
	```
	1) Scenario: log in # tests/feature/login.feature:5
   	✔ Before # conf.js:23
   	? When ... (whatever you defined as step in the feature file)
       Undefined. Implement with the following snippet:
	```

## 7. Creating a step definition (playwright)
- Copy & paste the proposed step definition from the terminal output (see 6.) to `tests/stepDefinition/context.js`
- Start implementation of the step as follows:
  - replace `function` with `async function`
  - delete `return 'pending'` line
  - add `await page.goto(url)` to visit oCIS instance (redirects to login page)

## 8. Run test again
- if you use localhost oCIS and have error: `page.goto: net::ERR_CERT_AUTHORITY_INVALID at https://localhost:9200/`,
	ignore certificate warning via additional `conf.js` line: 
    `global.context = await global.browser.newContext({ ignoreHTTPSErrors: true });`
- expected: test passed :-D

## 9. Use test generator
- in terminal: `npx playwright codegen https://ocis.ocis-traefik.released.owncloud.works`
- you can copy paste generated code from `Playwright Inspector` window to `context.js`
- run test. Expected: test passed :-D

---------

Go to the main room please!

---------

- or create a test yourself   
	use:  
  - locators https://playwright.dev/docs/selectors How choose: https://www.w3schools.com/cssref/css_selectors.asp
  - actions https://playwright.dev/docs/api/class-page
  - assertions https://playwright.dev/docs/test-assertions

## 10. Run test in different browsers 
- instead of `chromium` in `conf.js` use `webkit` or `firefox`
- Playwright launches safari even if you use linux or windows :-)

## 11. Configuration
- in `conf.js`
  - change `headless: true` to not show the browser
  - add `slowMo: 2000` each step of the test will be slower by 2000ms

## 12. Debugging and tracing
- tracing: https://playwright.dev/docs/trace-viewer
	- Set the trace path dir in conf: 
	```js
	global.browser = await chromium.launch({
        headless: false,
        tracesDir: 'tests/trace' 
   	});
	```
	- in section `Before` in `conf.js` after `browser.newContext()`, start tracing
	`await context.tracing.start({ screenshots: true, snapshots: true, sources: true})`
	- in section `After` in `conf.js` before `await global.page.close()`, finalize tracing
	`await context.tracing.stop({ path: 'tests/trace/trace.zip' });`
	- run test again
	- open https://trace.playwright.dev/ and select `zip` file from `tests/trace/trace.zip`
	- tracing provide a rich information. You can swich between `network` `console` `source`

<img width="1342" alt="Screenshot 2022-11-17 at 12 25 49" src="https://user-images.githubusercontent.com/84779829/202435943-cb868301-e15b-4e4d-9b61-fdd0d67cb7a7.png">

- debug: https://playwright.dev/docs/debug#pagepause
	- Turn on the browser again: `headless: false`
	- put await page.pause() to `context.js` after `await page.goto(url)` and run test
	- you should see Playwright inspector. you can go through all the steps of the test


# Ending:
 	- We finish our first introduction to e2e testing with playwright
 	- The next step will be a deeper dive to testing from the web repo and give an overview over existing steps (reusable)

---------

Go to the main room please!

---------

# Bonus: Create new test
- Scenario: user creates folder and checks api response after creating folder
- Hint: check api response after action https://playwright.dev/docs/api/class-page#page-wait-for-response

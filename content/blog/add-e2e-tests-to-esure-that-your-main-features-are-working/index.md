---
title: Add E2E tests to esure that your main features are working
date: "2021-03-06"
description: "Add e2e tests to ensure that your main features are working. This is usually cheap, fast and can prevent major problems."
---

Last night I had a problem with [TweetPik](https://tweetpik.com) where I made a small change and I didn't expect it to break the main feature - download a tweet as an image. But, this could be avoided if I had added a simple e2e test to ensure that the functionality was working as expected. So, I will briefly show you what I did this morning. Even if you don't have a deep tech skill, you will be able to understand and perhaps implement it on your side as well.

For e2e tests, I decided to use [Playwright Test](https://github.com/microsoft/playwright-test), which is a Microsoft open-source project that uses [Playwright](https://playwright.dev/) to run e2e tests because I had good experiences with it in another project and because it uses Typescript by default which matches my current codebase. Another good tool for that is [Cypress](https://www.cypress.io/).

I also decided to use [Folio](https://github.com/microsoft/folio) as test framework instead of other popular ones like Jest because it provides better support for Playwright's features, such as a nice CLI to run.

So, to install Playwright, you can use `npm` or `yarn`. Since I'm using Typescript, I added a babel transpiler.

**Using npm:**
```shell
$ npm i -D @playwright/test @babel/preset-typescript --dev  
```
**Using yarn:**
```shell
$ yarn add @playwright/test @babel/preset-typescript --dev
```

I created a folder in the root folder called `e2e` and I added the test file called `downloadTweetImage.test.ts`. I decided to organize my e2e tests by feature but you maybe want to organize in a different way like by page.

The test file `e2e/downloadTweetImage.test.ts` is quite simple:
```typescript
import { it, expect } from "@playwright/test";

it("downloads a tweet as PNG", async ({ page }) => {
  await page.goto("http://localhost:3000");
  await page.fill(
    "#search",
    "https://twitter.com/bruno__quaresma/status/1368231966714298368"
  );
  await page.press("#search", "Enter");
  await page.waitForSelector(".download");
  const [download] = await Promise.all([
    page.waitForEvent("download"),
    page.click(".download"),
    page.click(".png"),
  ]);

  expect(download).not.toBeUndefined();
});
```

Basically what I did was:
- Go to the page
- Fill the search input
- Press ENTER
- Expect the download button be displayed on the screen
- Wait for a download event
- Click on the download button
- Click on the PNG option
- Check if the downloaded content is not undefined

To run the tests I used Folio directly using `npx`:
```shell
$ npx folio --test-match e2e/*.ts
```
This runs all the tests across Chromium, Firefox and Webkit. Since the feature is not using any "not well supported API" I changed that to just run on Chromium to speed up things.
```shell
$ npx folio --param browserName=chromium --test-match e2e/*.ts
```
You can see more params [here](https://github.com/microsoft/playwright-test#run-the-test).

Also, you can add this as script on `package.json`:
```json
{
  "scripts": {
    "e2e": "npx folio --test-match e2e/*.ts"
  }
}
```
Now, to run the command you can do:

**Using npm:**
```shell
$ npm run e2e  
```
**Using yarn:**
```shell
$ yarn run e2e  
```

That is it. I hope you enjoyed the content and make you think about add e2e tests into your product covering the main features.

See ya!
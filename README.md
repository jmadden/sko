# Repo for DevOps with GitHub hands-on session

This repo has been created using twilio CLI for serverless. The only thing added is the dependency to `jest` and the `jest` test script. 

# Step 1 - Create a new repo based on this 

Create a new repo on your profile clicking on "Use this template" button

# Step 2 - Create a new branch 

Click on branch and create a new branch called `actions`

# Step 3 - Add a jest test script

Create a folder `__tests__` and insise this folder, create a file named `hello_world.test.js`. Copy the content from the file below: 

<details>
        <summary><b>Click here to view file contents to copy:</b></summary>
 
 ```javascript
 const Twilio = require('twilio');

describe('Test voice response TwiML', () => {
  beforeAll(() => {
    global.Twilio = Twilio;
  });

  it('returns "Hello World" TwiML response', (done) => {
    const tokenFunction = require('../functions/hello-world').handler;

    const callback = (err, twimlResponse) => {
      expect(twimlResponse.toString()).toBe(
        '<?xml version="1.0" encoding="UTF-8"?><Response><Say>Hello World!</Say></Response>'
      );
      done();
    };

    tokenFunction(null, {}, callback);
  });
});
```
 </details>
 
 # Step 4 - Add GitHub Action for testing
 
Create a new folder `.github/workflows`. In this folder create a new file named `test.yml`. Paste the content from the snippet below: 
 
 <details>
        <summary><b>Click here to view file contents to copy:</b></summary>
 
 ```yaml
 name: Twilio Serverless testing

on: [pull_request, push]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [macos-latest]
        node-version: [10]

    steps:
      - name: Checkout code
        uses: actions/checkout@v1
      - name: Use Node.js version ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
      - name: npm install, build, and test
        run: |
          npm install
          npm run jest
        env:
          CI: true
 ```
 
 </details>
 
 After committing this file, open the action and see the test running. 
 
 # Step 5 - Add Secrets
 
 * In the repo click on Settings 
 * Click on "Secrets" in the lef side bar
 * Click on "New repository secret"
 * Add two secrets: 
   * `TWILIO_ACCOUNT_SID`
   * `TWILIO_AUTH_TOKEN`
 
# Step 6 - Add script for deployment 
 
Add a file named `deploy.yaml` in the folder `.github/workflows`. Paste the content of the snippet below and commit: 
<details>
        <summary><b>Click here to view file contents to copy:</b></summary>

```yaml
name: Deployment to Twilio Serverless

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [macos-latest]
        node-version: [10]

    steps:
      - name: Checkout code
        uses: actions/checkout@v1
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
      - name: npm install and deploy
        run: |
          npm install
          npm run deploy -- --account-sid ${{secrets.TWILIO_ACCOUNT_SID}} --auth-token ${{secrets.TWILIO_AUTH_TOKEN}} --override-existing-project
        env:
          CI: true

```
</details>
        
# Step 7 - Create a new deployment
        
* Click on "Pull request"
* Make sure you select your own main branch as base (by default GitHub may attempt to create a pull request to the original repo you forked from)
* Wait for the test to be executed 
* Click on merge 

If you go to the Actions tab, you will see the `test` and `deploy` workflow running in parallel

# Step 8 - Add Secrets for Twilio SMS 

Open the secret page in the repo settings and create the following secrets: 

* `TWILIO_ACCOUNT_SID`: this is your Twilio account SID or your API key
* `TWILIO_AUTH_TOKEN`: this is your Twilio auth token or your API secret
* `TWILIO_SMS_API_KEY`: this is an API to send SMS (create one at https://www.twilio.com/console/sms/project/api-keys)
* `TWILIO_SMS_API_SECRET`: this is the secret for the API key created above
* `TWILIO_SMS_FROM`: Phone number in your Twilio account to send the SMS from
* `TWILIO_SMS_TO`: Phone number to send the SMS to

# Step 9 - Add Twilio SMS notification 

Open the `.github/workflows/deploy.yaml` and add the content of the snippets below to the file (careful with the two blank spaces before `notify`): 
<details>
        <summary><b>Click here to view file contents to copy:</b></summary>
        
```yaml
  notify:
    runs-on: ubuntu-latest
    needs: deploy

    steps:
      - name: Send an SMS through Twilio
        uses: twilio-labs/actions-sms@v1
        with:
          fromPhoneNumber: ${{ secrets.TWILIO_SMS_FROM }}
          toPhoneNumber: ${{ secrets.TWILIO_SMS_TO }}
          message: '🚢 Deployment successful 🎉'
        env:
          TWILIO_ACCOUNT_SID: ${{ secrets.TWILIO_ACCOUNT_SID }}
          TWILIO_API_KEY: ${{ secrets.TWILIO_SMS_API_KEY }}
          TWILIO_API_SECRET: ${{ secrets.TWILIO_SMS_API_SECRET }}
```
</details>

Commit this to the `main` branch directly. After the deploy is executed, you should be receiving a message

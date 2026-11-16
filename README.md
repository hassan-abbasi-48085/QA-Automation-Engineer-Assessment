# QA Automation Engineer Assessment

Solutions to the three tasks in the QA Automation Engineer assessment:

1. **API automation:** create a user on the GoREST API, then read it back and check it.
2. **UI automation:** write manual test cases for a web page and automate one of them.
3. **Kafka automation (optional):** write manual test cases for a Kafka producer and consumer, and automate one of each.

Each task is a separate Node.js project with its own dependencies, tests and detailed README. The original brief is in [PROBLEM STATEMENT.txt](PROBLEM%20STATEMENT.txt).

## At a glance

| Task | Asked for | Delivered | Stack |
|---|---|---|---|
| [1. API testing](01_GO%20REST/qa-assessment-task1-extended/DOCS/README.md) | POST a user to GoREST, find it with GET, and check its properties | 14 Gherkin scenarios. Two cover the required create-and-get flow. The rest cover authentication, input validation and the full lifecycle (PATCH, DELETE). | Cucumber.js, Chai, native `fetch` |
| [2. UI testing](02_LOGIN%20TEST/Project%20Files/README.md) | 3 manual test scenarios for any web page, and 1 of them automated | 8 manual test cases for a login page, all 8 automated. TC-LOGIN-01 is the required one. | Playwright, playwright-bdd, Page Object Model |
| [3. Kafka testing](03_KAFKA%20Automation%20TESTING/PROJECT%20FILES/README.md) | 6 manual producer and consumer scenarios, with 1 producer and 1 consumer scenario automated | 6 scenarios in Gherkin. KAFKA-P-01 (producer) and KAFKA-C-01 (consumer) are each automated twice, once in Cucumber and once in Mocha + Chai, against a real Kafka broker and Node-RED flows. | Kafka 3.9.1 (KRaft), Node-RED 4.0.9, kafkajs, Mocha, Chai, Cucumber, Docker Compose |

## Repository layout

```
.
├── PROBLEM STATEMENT.txt                  The original assessment brief
│
├── 01_GO REST/                            Task 1: API automation
│   ├── PROBLEM STATEMENT.txt
│   ├── GoREST_Task1_Workflow.html         Workflow write-up (open in a browser)
│   └── qa-assessment-task1-extended/      Test project (README in DOCS/)
│
├── 02_LOGIN TEST/                         Task 2: UI automation
│   ├── PROBLEM STATEMENT.txt
│   ├── WORKFLOW.html                      Workflow write-up (open in a browser)
│   └── Project Files/                     Test project and manual test cases
│
└── 03_KAFKA Automation TESTING/           Task 3: Kafka automation
    ├── PROBLEM STATEMENT.txt
    ├── kafka-e2e-workflow.html            Workflow write-up (open in a browser)
    ├── TerminalCommands.txt               Running Kafka and Node-RED on Windows without Docker
    └── PROJECT FILES/                     Kafka stack, Node-RED flows and tests
```

Each workflow write-up covers the problem, the approach, the code structure, how to run the tests and the results. The `*_ThisPC.html` files have the same content, with file paths set for the author's machine.

## Prerequisites

| Requirement | Needed for |
|---|---|
| Node.js 20 or newer | All tasks. Task 2 needs 20+. Tasks 1 and 3 also run on Node 18. |
| A GoREST access token (free at [gorest.co.in/consumer/login](https://gorest.co.in/consumer/login)) | Task 1 |
| Docker with the Compose v2 plugin, and free ports `9092` and `1880` | Task 3 |
| Internet access | Tasks 1 and 2 (the tests call live sites) |

## Running the tests

Run every command from the repository root. Each project installs on its own.

### Task 1: GoREST API

PowerShell:

```powershell
cd "01_GO REST/qa-assessment-task1-extended"
npm install
$env:GOREST_TOKEN = "<your-gorest-token>"
npm test
```

Bash:

```bash
cd "01_GO REST/qa-assessment-task1-extended"
npm install
read -rsp "GoREST token: " GOREST_TOKEN && echo && export GOREST_TOKEN
npm test
```

`npm run test:task1` runs only the two scenarios the task asks for. Reports are written to `reports/cucumber-report.html` and `reports/cucumber-report.json`.

### Task 2: Login UI

```bash
cd "02_LOGIN TEST/Project Files"
npm ci
npx playwright install chromium
npm test               # all 8 scenarios, headless Chromium
npm run test:smoke     # only TC-LOGIN-01, the required scenario
npm run test:headed    # watch the browser
npm run report         # open the Playwright HTML report
```

### Task 3: Kafka and Node-RED

```bash
cd "03_KAFKA Automation TESTING/PROJECT FILES"
docker compose up -d --build --wait    # start Kafka, create the topics, start Node-RED
npm ci
npm test                               # Cucumber features, then Mocha specs
docker compose down -v                 # stop the stack and delete its data
```

`npm run test:unit` (15 contract tests) and `npm run lint` run without a broker. The Node-RED editor is at <http://localhost:1880> while the stack is up. To run without Docker on Windows, follow `03_KAFKA Automation TESTING/TerminalCommands.txt`.

## Test coverage

### Task 1: GoREST users API

| Feature file | What it checks |
|---|---|
| `01-create-and-get.feature` | **Required task.** POST returns 201 and an ID. GET with that ID returns 200, and `id`, `name`, `email`, `gender` and `status` match what was sent. |
| `02-authentication.feature` | POST with no token, or with an invalid token, returns 401. |
| `03-validation.feature` | POST returns 422 for a taken email, an invalid email, gender or status, and an empty body. |
| `04-lifecycle.feature` | GET returns exactly the five expected fields. PATCH updates the user. DELETE returns 204, and later requests for that user return 404. |
| `05-credentials.feature` | A missing or blank token stops the test with a clear message before any request is sent. |

### Task 2: Login page

Site under test: [practicetestautomation.com/practice-test-login](https://practicetestautomation.com/practice-test-login/), which publishes its own demo credentials. The full manual test cases are in [MANUAL_TEST_CASE.md](02_LOGIN%20TEST/Project%20Files/MANUAL_TEST_CASE.md).

| ID | Scenario | Type |
|---|---|---|
| TC-LOGIN-01 | Successful login with valid credentials (**required automated scenario**) | Positive |
| TC-LOGIN-02 | Valid username, wrong password | Negative |
| TC-LOGIN-03 | Both fields blank | Negative |
| TC-LOGIN-04 | Invalid username, valid password | Negative |
| TC-LOGIN-05 | Valid username, blank password | Negative |
| TC-LOGIN-06 | Password field hides the typed value | Security |
| TC-LOGIN-07 | Logging out returns to the login page | Positive |
| TC-LOGIN-08 | Valid login after a failed attempt | Recovery |

### Task 3: Kafka producer and consumer

The system under test is a Node-RED producer flow that takes `POST /orders` and publishes an `OrderCreated` event to `qa.orders.v1`. A Node-RED consumer flow reads that topic and writes `OrderProcessed` to `qa.orders.processed.v1`, or sends invalid events to the dead-letter topic `qa.orders.dlq.v1`. The manual steps and observed results are in [manual-test-scenarios.md](03_KAFKA%20Automation%20TESTING/PROJECT%20FILES/docs/manual-test-scenarios.md).

| ID | Scenario | Automated |
|---|---|---|
| KAFKA-P-01 | Producer publishes a valid event | **Yes** (Cucumber and Mocha) |
| KAFKA-P-02 | Producer rejects an order with a missing or invalid field | Manual |
| KAFKA-P-03 | Producer handles an unavailable broker and recovers without a restart | Manual |
| KAFKA-C-01 | Consumer processes a valid event and produces a downstream result | **Yes** (Cucumber and Mocha) |
| KAFKA-C-02 | Consumer sends a malformed event to the dead-letter topic and carries on | Manual |
| KAFKA-C-03 | Consumer restarts and resumes from its committed offset | Manual |

## Approach

- **Test cases are written in Gherkin.** In all three tasks, the manual test cases and the automated tests use the same Given/When/Then text, so a reviewer reads one version of each case.
- **Tests clean up after themselves.** Task 1 deletes every user it creates, even after a failure. Task 3 gives each run a unique ID, which it uses as the correlation ID and test consumer group name, and deletes that group afterwards. Runs never interfere with each other.
- **Test code is layered.** Task 2 uses the Page Object Model, so step definitions never contain selectors. Task 3 applies the same idea to messaging with client objects (HTTP client, Kafka publisher, Kafka subscriber) and a test data builder.
- **No fixed sleeps.** Playwright's assertions retry until they pass or time out. Every Kafka wait has a time limit and, if it runs out, names what it was waiting for.
- **Credentials come from the environment.** The GoREST token is read only from `GOREST_TOKEN`, and failure messages never print request headers.
- **The tests were shown to fail.** For Tasks 2 and 3, an expected value was deliberately changed, or the services were stopped, to confirm that the tests then fail with a clear message.
- **CI is ready for Task 3.** `PROJECT FILES/ci/github-actions-e2e.yml` lints the code, starts the stack, runs every test and uploads the reports. To enable it, copy it into `.github/workflows/`.

## Results

Last recorded runs, taken from each task's workflow write-up:

| Task | Result | Run on |
|---|---|---|
| 1 | 14 scenarios and 100 steps passed | 14 Nov 2026 · Windows 11, Node 18.12.1 |
| 2 | 8 scenarios passed in 11.1 s | 14 Nov 2026 · Windows 11, Node 22.23.5, Chromium |
| 3 | Cucumber: 2 scenarios and 24 steps passed. Mocha: 2 integration specs and 15 unit tests passed. | 14 Nov 2026 · Windows 11, Docker 27.4, Node 18.12.1 |

Each project's README has its full verification notes.

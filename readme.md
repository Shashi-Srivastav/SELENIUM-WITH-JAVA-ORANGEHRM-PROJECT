# Selenium with Java: OrangeHRM Practice Project

This repository is a small browser-automation practice project for the public OrangeHRM demo site. It uses Java, Selenium WebDriver, TestNG annotations and Maven dependencies. It contains a Page Object Model example for login, an additional direct-locator test class with several exploratory scenarios, and AutoIt helpers for native file-picker dialogs.

> **Repository accuracy note:** The README describes the checked-in source. Several scenarios listed in `Notes` are not currently implemented as active tests. In `Demo.OrangeHRM`, most methods are explicitly disabled; only `applyLeave` is enabled. Maven's default test discovery also does not select `OrangeHRM` by its class name. This README distinguishes the documented ideas from what a normal Maven run actually executes.

## What is in the repository

```text
SELENIUM-WITH-JAVA-ORANGEHRM-PROJECT/
├── src/
│   ├── main/java/SeleniumPractice/SeleniumPractice/
│   │   └── App.java
│   └── test/java/
│       ├── Demo/
│       │   └── OrangeHRM.java
│       ├── PageObjects/
│       │   ├── DashboardPage.java
│       │   └── LoginPage.java
│       ├── SeleniumPractice/SeleniumPractice/
│       │   └── AppTest.java
│       └── Tests/
│           └── LoginTest.java
├── test-output/                  # Previously generated TestNG reports
├── target/                       # Previously generated compiled classes
├── AddImageOrangeHRM.au3         # AutoIt source for image-picker dialog
├── AddImageOrangeHRM.exe         # Compiled Windows helper
├── File.csv                      # Sample CSV upload data
├── FileUploadOrangeHRM.au3       # AutoIt source for CSV-picker dialog
├── FileUploadOrangeHRM.exe       # Compiled Windows helper
├── Notes                         # Project notes and intended scenarios
├── photo1.jpg                    # Sample image asset
└── pom.xml                       # Maven project and dependency definition
```

`target/` and `test-output/` are generated artifacts checked into the repository; the `src/` tree and root-level assets are the main editable project inputs.

| Path | Purpose |
| --- | --- |
| `pom.xml` | Maven coordinates and Selenium, TestNG, and JUnit dependencies. It does not configure a TestNG suite, browser, Java source level, or reporting plugin. |
| `src/test/java/Tests/LoginTest.java` | Active TestNG login tests using page objects: invalid credentials followed by valid credentials. |
| `src/test/java/PageObjects/LoginPage.java` | Page object for login fields, submit button, and invalid-credentials message. Uses Selenium `PageFactory` and `@FindBy`. |
| `src/test/java/PageObjects/DashboardPage.java` | Small page object that reads the current browser title. |
| `src/test/java/Demo/OrangeHRM.java` | Direct-locator practice class with login, employee, file upload, pagination, deletion, and leave scenarios. Most tests are disabled. |
| `src/main/java/SeleniumPractice/SeleniumPractice/App.java` | Maven archetype placeholder that prints `Hello World!`; not part of the browser tests. |
| `src/test/java/SeleniumPractice/SeleniumPractice/AppTest.java` | JUnit 3 archetype placeholder test that asserts `true`. |
| `Notes` | Base URL, intended test-case descriptions, POM explanation, and AutoIt setup notes. These are notes, not executable configuration. |
| `File.csv`, `photo1.jpg` | Sample data/image assets referenced by the AutoIt scripts via absolute paths. |
| `AddImageOrangeHRM.au3`, `FileUploadOrangeHRM.au3` | AutoIt scripts that populate and submit a Windows file-open dialog. |
| `AddImageOrangeHRM.exe`, `FileUploadOrangeHRM.exe` | Compiled Windows AutoIt helpers, launched by the exploratory employee/file-upload code. |
| `test-output/`, `target/` | Checked-in generated TestNG reports and compiled output. These are not source-of-truth configuration and may be stale. |

The repository does not contain a configuration properties file, external test-data framework, shared driver factory, base test class, screenshot listener, logging framework, or dedicated report plugin. Browser creation and test data are directly in the test classes.

## Framework and architecture

### Maven dependencies

The POM declares:

- Selenium Java `4.18.1` for browser automation.
- TestNG `7.7.1` with test scope for `@BeforeTest`, `@Test`, `@AfterTest`, and assertions in the test sources.
- JUnit `3.8.1` with test scope for the generated `AppTest` class.

There is no explicit Maven Surefire configuration or `testng.xml`. The tests rely on Maven's normal test lifecycle and class-name discovery. The POM also does not pin Maven compiler source/target or a Java release, so the effective language level depends on the installed Maven/compiler setup.

### Page Object Model used by `LoginTest`

`LoginTest` creates one `ChromeDriver` in `setup()`, maximizes the window, navigates to the OrangeHRM demo login URL, and sets a 60-second implicit wait. The test methods instantiate `LoginPage` around the same driver.

`LoginPage` centralizes locators for username (`name="username"`), password (`name="password"`), submit button, and invalid-credentials text. It exposes methods to type values, submit, and read the error message. `DashboardPage` wraps the same driver and exposes `getPageTitle()`; it does not locate or assert dashboard elements.

This provides a basic separation between login selectors/actions and test assertions. There is no shared page superclass, navigation abstraction, or driver lifecycle utility; those responsibilities remain in each test class.

## What the active tests do

### `Tests.LoginTest`

Both TestNG test methods are enabled and have explicit priorities:

1. `doLoginWithInvalidCredential` enters `Admin` / `1234`, submits the form, and asserts that the alert text equals `Invalid credentials`.
2. `loginTestWithValidCredential` then enters `Admin` / `admin123`, submits, and asserts that `driver.getTitle()` is `OrangeHRM`.

The class uses TestNG's `@BeforeTest` and `@AfterTest`, so these methods share the browser created for that test configuration. The invalid attempt is expected to leave the browser on the login page for the next test. Teardown waits five seconds, then calls both `close()` and `quit()`. The test does not log out after successful login; the browser is closed during teardown.

### `Demo.OrangeHRM`

`setup()` starts Chrome and navigates to the login page, but Maven's default Surefire class patterns do not include the class name `OrangeHRM`. To select it explicitly use the command shown below. Within the class, test priorities 1 through 8 (invalid login, valid login, add employee, searches, upload, delete, listing) are all annotated `enabled=false`. The `applyLeave` method has `enabled=true`; it logs in, opens Leave > Apply, selects an option whose text contains `CAN`, enters a fixed date and comment, submits, waits, and invokes logout.

When explicitly run, `applyLeave` depends on the current state and data of the remote demo site. It has no assertion that verifies a leave request was created, and the leave date is fixed to `2024-08-04`, which is in the past relative to the current date. It is therefore best understood as a practice script, not a reliable acceptance test.

The other scenarios in `OrangeHRM` show the intended or attempted interactions:

- Add an employee and check for the `Personal Details` heading.
- Search by employee name or ID and inspect result text/cells.
- Upload an employee image or CSV using an AutoIt executable.
- Delete an employee and look for `No Records Found`.
- Iterate through employee-list pages and print a column value.

These methods are disabled and should not be interpreted as validated behavior. `Notes` also describes employee-add and search scenarios as intended cases, but their current code is disabled.

## Test execution flow

For `mvn test`, Maven resolves dependencies and compiles the project, then its test provider discovers classes by its default naming patterns. In this repository the likely selected test classes are `Tests.LoginTest` (TestNG) and `SeleniumPractice.SeleniumPractice.AppTest` (JUnit); `Demo.OrangeHRM` is not selected by its name. The first opens Chrome and runs the two prioritized login tests; the JUnit archetype test simply checks `true`.

The exact test provider and discovery result can vary with Maven/Surefire versions because the POM does not pin or configure the Surefire plugin. The checked-in `test-output` folder reflects a prior execution and is not proof that a fresh run will pass against the live demo site.

## Prerequisites

- A supported JDK and Maven installed and available on `PATH`.
- Google Chrome installed. `LoginTest` constructs `ChromeDriver` directly. Selenium 4.18 includes Selenium Manager integration to locate/manage a compatible driver when possible, but a working browser/driver setup and network access may still be needed in a given environment.
- Network access to `https://opensource-demo.orangehrmlive.com/web/index.php/auth/login` and Maven repositories.
- For the AutoIt file-dialog examples only: Windows, the checked-in helper executable, and paths matching the paths hard-coded in the `.au3` scripts and Java code.

The demo credentials `Admin` / `admin123` are embedded in the source. They are public demo credentials, not secrets or project configuration.

## Build and run

Run from the repository root (the directory containing `pom.xml`):

```bash
# Compile and run Maven-discovered tests
mvn test

# Clean generated files, then compile and run tests
mvn clean test

# Build/package without running tests
mvn -DskipTests package

# Explicitly ask Surefire to run the exploratory OrangeHRM class
mvn -Dtest=OrangeHRM test

# Select the login test class explicitly
mvn -Dtest=LoginTest test
```

`mvn test` should not be assumed to run `Demo.OrangeHRM`; its class name does not match the usual Maven Surefire test patterns. An explicit `-Dtest=OrangeHRM` selection bypasses that default class-name filter, though execution still depends on Maven's TestNG provider and the current remote application. Because `applyLeave` is enabled, explicitly selecting the class can submit a leave request to the shared public demo account. Employee management and upload scenarios remain disabled unless their annotations are changed.

If driver startup fails, check Chrome installation and Selenium Manager's ability to obtain/find a compatible driver. The project does not include a manual driver binary, browser selection setting, or driver path configuration.

## Reports and generated files

TestNG output exists under `test-output/` in the checked-in repository, including HTML and XML report files. It was generated by an earlier run and can be overwritten or become outdated. Maven's normal build output goes under `target/`. There is no configured Extent Report, Allure, custom report listener, or automatic screenshot-on-failure feature in the source.

## How to add a test

For a login-focused test:

1. Add a public test method in `src/test/java/Tests/LoginTest.java` (or a new test class with a name Maven discovers, such as `*Test.java`).
2. Mark it with TestNG `@Test`; use `priority` only when ordering is required.
3. Reuse `LoginPage` for login fields and actions. Add a page method or a new page object when a page needs reusable locators and operations.
4. Keep expected results in assertions in the test method. Prefer assertions against a stable visible result rather than only a title or fixed delay.
5. Run the class explicitly with `mvn -Dtest=YourTestClass test` while developing, then run `mvn test` to check Maven discovery.

The current test setup shares a single browser across methods in `LoginTest`; tests that alter session state should either restore the expected state or introduce per-test setup/teardown. The project currently uses implicit waits plus several `Thread.sleep` calls in the exploratory class; avoid adding fixed sleeps when a condition-based wait can express the needed state more reliably.

## Important implementation limits

- The base URL is duplicated in Java classes rather than loaded from configuration.
- Credentials, test names, employee ID, leave date, and comments are hard-coded.
- Locators are embedded in the page objects and exploratory test class; there is no locator/data configuration layer.
- `OrangeHRM` file-dialog scripts contain absolute paths such as `C:\photo1.jpg` and `C:\File.csv`. Its Java code also launches executables from a user-specific `C://Users//ASUS//Desktop//...` location. Those paths will not work on most machines without editing.
- The AutoIt scripts interact with a Windows native file picker by window title/control identifiers. This approach is OS- and dialog-dependent and cannot be used in headless Linux/macOS environments. For a normal HTML `<input type="file">`, Selenium `sendKeys` with an absolute file path is generally simpler, when the application exposes the input.
- `OrangeHRM` has `logOut()` implemented by clicking the dropdown and selecting the fourth matching link. This relies on ordering and is brittle. `LoginTest` does not call it.
- The code uses long implicit waits and fixed sleeps in places, and does not use explicit waits, retry logic, screenshots, or test isolation.
- The public demo site and its records can change independently of this repository, so credentials, selectors, employee records, and the outcome of state-changing actions may stop matching expectations.

## Scope and provenance

This guide is based on the repository source and assets available on its `master` branch when inspected. The original repository contains no README documenting a different supported workflow. Where Maven discovery or remote-site behavior depends on external tool versions or current site state, this README identifies that dependency instead of asserting a guaranteed result.

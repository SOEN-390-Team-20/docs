# JeVaisBienAller Testing Plan

Written documentation, even comments on functions tend to go stale, becoming irrelevant, but along with the source code, tests are up-to-date documentation of the usage of our code. Good test coverage also allows us to be more agile: we can refactor big portions of the code without fear, since we can trust that our tests will pick up breaking changes. It is therefore vital to have a proper testing plan.

Our testing plan consists of a series of backend tests, frontend tests and finally some integration tests to ensure that our software meets our design. We consider our backend and frontend tests as our unit tests, since they test our app's components as we implement them. Our integration tests is more closely related to our user stories.

## Backend tests
\[[Files](https://github.com/SOEN-390-Team-20/Team-20-SOEN-390/tree/main/tests)\] \[[View on CircleCI](https://app.circleci.com/pipelines/github/SOEN-390-Team-20/Team-20-SOEN-390/201/workflows/7e55f7ac-6b90-4ee8-b639-13874cbfd3f5/jobs/405)]

<details>
  <summary>Console output</summary>

  ```bash
  > ***************-app@0.0.0 test:only
  > cross-env NODE_ENV=test jest --verbose --runInBand

   PASS  tests/users.test.js
    REST API requests on /api/users/ (expects test users to be added)
      ✓ OLD: POST /api/users : TEST_PATIENT2 can register (355 ms)
      ✓ JWT: POST /api/users : TEST_PATIENT2 can register (449 ms)
      ✓ PUT /api/users/:id : TEST_PATIENT2 can be updated (161 ms)
      ✓ DELETE /api/users/:id : TEST_PATIENT2 can be deleted (231 ms)
      ✓ GET /api/users/:id : TEST_PATIENT1 can fetch their information (153 ms)

   PASS  tests/login.test.js
    OLD: REST API requests on /api/login (expects test users to be added)
      ✓ POST /api/login : TEST_PATIENT1 can login (105 ms)
      ✓ POST /api/login : TEST_PATIENT1 cannot login with bad credentials (75 ms)
    JWT Token: REST API requests on /api/login (expects test users to be added)
      ✓ POST /api/login : TEST_PATIENT1 can login (138 ms)
      ✓ POST /api/login : TEST_PATIENT1 cannot login with bad credentials (73 ms)

   PASS  tests/oldapi.test.js
    (Old Api) REST API requests on /rest/api/
      ✓ POST /rest/api/add-user : TEST_PATIENT1 can register (809 ms)
      ✓ POST /rest/api/login : TEST_PATIENT1 can login (747 ms)
      ✓ POST /rest/api/login : TEST_PATIENT1 cannot login with bad credentials (722 ms)

   PASS  tests/healthform.test.js
    REST API request on /api/forms
      ✓ POST /api/forms is successful (1510 ms)

   PASS  tests/static.test.js
    Static routes tests
      ✓ GET / : React app is accessible (32 ms)
      ✓ GET /health : is accessible (5 ms)
      ✓ GET /version: is accessible (2 ms)

  Test Suites: 5 passed, 5 total
  Tests:       16 passed, 16 total
  Snapshots:   0 total
  Time:        14.165 s
  Ran all test suites.
  ```
  
</details>

For the backend tests, we have tests for our api endpoints. Our backend tests are also integration tests, technically speaking, because it uses a live database (jevaisbienaller-test). We could be mocking `MongoDB` and `mongoose` (like with [mockingoose](https://www.npmjs.com/package/mockingoose)) but we found that there was more business value to test the api directly with the live database for now. We can assert for status code and for an expected json output from the api with a simple syntax using [SuperTest](https://www.npmjs.com/package/supertest) and [Jest](https://jestjs.io/) together. Our coverage is complete for important endpoints like the login and the registration.

<details>
  <summary>Example test</summary>
  
  ```js
   test('OLD: POST /api/users : TEST_PATIENT2 can register', async () => {
    // Precondition: user should not already exist
    // This is a call to the live database through our controller
    const isUserExist = await User.findOne({ email: TEST_PATIENT2.email });
    if (isUserExist) {
      await User.findByIdAndDelete(isUserExist.id);
    }

    // Register using the patient payload
    await api
      .post('/api/users')
      .send(TEST_PATIENT2)
      .expect(200)
      .expect('Content-Type', /application\/json/);

    // Postcondition: user should exist in the mongodb database
    const addedUser = await User.findOne({ email: TEST_PATIENT2.email });
    expect(addedUser.email).toContain(TEST_PATIENT2.email);
  });
  ```
  
</details>

## Frontend tests
\[[Files](https://github.com/SOEN-390-Team-20/Team-20-SOEN-390/tree/main/client/src/views/__test__)] \[[View on CircleCI](https://app.circleci.com/pipelines/github/SOEN-390-Team-20/Team-20-SOEN-390/201/workflows/955e87c7-de6f-454d-bd8d-bdacb953a07d/jobs/402)]

<details>
  <summary>Console Output</summary>
  
  ```bash
  
  > frontend@0.1.0 test
  > react-scripts test

   PASS  src/views/__test__/CheckinScreen.test.js
   PASS  src/views/__test__/LoginScreen.test.js

  Test Suites: 2 passed, 2 total
  Tests:       2 passed, 2 total
  Snapshots:   0 total
  Time:        3.248 s
  Ran all test suites.
  ```
  
</details>

Here we use the [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/). Our frontend tests are pretty basic compared to our backend. This is definitely an area where we can improve, but we have little component reuse at this point. It should be noted that the integration tests test the frontend more in-depth than the unit tests we have here.

<details>
  <summary>Example test</summary>
  We basically assert that the component renders the expected text.
  
  ```js
  it("should have initial title 'Login'", async () => {
  render(<MockLogin />);
  const title = screen.getByText(/login/i);
  expect(title).toBeInTheDocument();
  });
  ```
</details>

## Integration tests
\[[Files](https://github.com/SOEN-390-Team-20/Team-20-SOEN-390/tree/main/client/src/views/__test__)] \[[View on CircleCI](https://app.circleci.com/pipelines/github/SOEN-390-Team-20/Team-20-SOEN-390/201/workflows/7e55f7ac-6b90-4ee8-b639-13874cbfd3f5/jobs/406)]

<details>
  <summary>Console Output</summary>
  
  ```bash
  > ***************-app@0.0.0 cy:run /root/project
  > start-server-and-test start:test http://localhost:3001 cypress:run

  1: starting server using command "npm run start:test"
  and when url "[ 'http://localhost:3001' ]" is responding with HTTP status code 200
  running tests using command "npm run cypress:run"


  > ***************-app@0.0.0 start:test /root/project
  > cross-env NODE_ENV=test node ./bin/www


  > ***************-app@0.0.0 cypress:run /root/project
  > cypress run



  [465:0223/181137.467918:ERROR:bus.cc(392)] Failed to connect to the bus: Failed to connect to socket /var/run/dbus/system_bus_socket: No such file or directory
  [465:0223/181137.469703:ERROR:bus.cc(392)] Failed to connect to the bus: Could not parse server address: Unknown address type (examples of valid types are "tcp" and on UNIX "unix")
  [465:0223/181137.469736:ERROR:bus.cc(392)] Failed to connect to the bus: Could not parse server address: Unknown address type (examples of valid types are "tcp" and on UNIX "unix")
  [653:0223/181137.511098:ERROR:gpu_init.cc(453)] Passthrough is not supported, GL is swiftshader, ANGLE is 

  ====================================================================================================

    (Run Starting)

    ┌────────────────────────────────────────────────────────────────────────────────────────────────┐
    │ Cypress:        9.5.0                                                                          │
    │ Browser:        Electron 94 (headless)                                                         │
    │ Node Version:   v14.16.0 (/usr/local/bin/node)                                                 │
    │ Specs:          2 found (LoginPage.spec.js, SignUpPage.spec.js)                                │
    └────────────────────────────────────────────────────────────────────────────────────────────────┘


  ────────────────────────────────────────────────────────────────────────────────────────────────────

    Running:  LoginPage.spec.js                                                               (1 of 2)
  [465:0223/181138.779088:ERROR:bus.cc(392)] Failed to connect to the bus: Could not parse server address: Unknown address type (examples of valid types are "tcp" and on UNIX "unix")
  Browserslist: caniuse-lite is outdated. Please run:
  npx browserslist@latest --update-db

  Why you should do it regularly:
  https://github.com/browserslist/browserslist#browsers-data-updating


    Test Login page
      ✓ Can login with proper user (2306ms)
      ✓ Cannot login with wrong user (1598ms)


    2 passing (4s)


    (Results)

    ┌────────────────────────────────────────────────────────────────────────────────────────────────┐
    │ Tests:        2                                                                                │
    │ Passing:      2                                                                                │
    │ Failing:      0                                                                                │
    │ Pending:      0                                                                                │
    │ Skipped:      0                                                                                │
    │ Screenshots:  0                                                                                │
    │ Video:        true                                                                             │
    │ Duration:     3 seconds                                                                        │
    │ Spec Ran:     LoginPage.spec.js                                                                │
    └────────────────────────────────────────────────────────────────────────────────────────────────┘


    (Video)

    -  Started processing:  Compressing to 32 CRF                                                     
    -  Finished processing: /root/project/cypress/videos/LoginPage.spec.js.mp4             (0 seconds)


  ────────────────────────────────────────────────────────────────────────────────────────────────────

    Running:  SignUpPage.spec.js                                                              (2 of 2)
  [465:0223/181146.708111:ERROR:bus.cc(392)] Failed to connect to the bus: Could not parse server address: Unknown address type (examples of valid types are "tcp" and on UNIX "unix")


    Test SignUp page
      ✓ Can Register a new user (5896ms)


    1 passing (6s)


    (Results)

    ┌────────────────────────────────────────────────────────────────────────────────────────────────┐
    │ Tests:        1                                                                                │
    │ Passing:      1                                                                                │
    │ Failing:      0                                                                                │
    │ Pending:      0                                                                                │
    │ Skipped:      0                                                                                │
    │ Screenshots:  0                                                                                │
    │ Video:        true                                                                             │
    │ Duration:     5 seconds                                                                        │
    │ Spec Ran:     SignUpPage.spec.js                                                               │
    └────────────────────────────────────────────────────────────────────────────────────────────────┘


    (Video)

    -  Started processing:  Compressing to 32 CRF                                                     
    -  Finished processing: /root/project/cypress/videos/SignUpPage.spec.js.mp4             (1 second)


  ====================================================================================================

    (Run Finished)


         Spec                                              Tests  Passing  Failing  Pending  Skipped  
    ┌────────────────────────────────────────────────────────────────────────────────────────────────┐
    │ ✔  LoginPage.spec.js                        00:03        2        2        -        -        - │
    ├────────────────────────────────────────────────────────────────────────────────────────────────┤
    │ ✔  SignUpPage.spec.js                       00:05        1        1        -        -        - │
    └────────────────────────────────────────────────────────────────────────────────────────────────┘
      ✔  All specs passed!                        00:09        3        3        -        -        -  
  ```
</details>

Our integration tests cover the following stories:
* User can register
* User can login

by interacting directly with a test version of our website using [Cypress](https://www.cypress.io/). It also asserts that the frontend follows the design of the UI prototype.

<details>
  <summary></summary>
</details>

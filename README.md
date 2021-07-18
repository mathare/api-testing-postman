[![Continuous Integration Status](https://github.com/mathare/api-testing-postman/actions/workflows/ci.yml/badge.svg)](https://github.com/mathare/api-testing-postman/actions)

# REST API Testing with Postman

## Overview
This project provides an example for testing REST APIs with Postman. It can be used to kickstart testing of other REST APIs with minimal changes to the project.

NB This is not a complete implementation of a Postman test suite for the target API. It is an example of how to structure a Postman test suite but only a subset of the requests (tests) have been added.

### Why use Postman?
[Postman](https://www.postman.com/) is a popular and easy-to-use API testing tool. It is simple to build & send requests and examine the responses, making it popular for exploratory and manual testing of APIs. However, Postman is capable of much more and is often overlooked as an automated API testing tool. 

Postman tests are written using JavaScript and the [Chai](https://www.chaijs.com/api/bdd/) assertion library. There is no support for other languages, and the Postman syntax can be a little strange if you're not used to it, even if you're an experienced JavaScript coder but at the same time there are a number of code snippets provided within Postman to help you create your tests. Often people use the code snippets to perform simple response code checks but may not go beyond this. I want to show how more detailed tests can be written - and easily maintained - within Postman. 

#### Pricing

Postman has a free tier to their pricing plan which allows up to 3 users to access the basic functionality. This tier is adequate for this test suite i.e. the tests can be run without cost. That same plan will also support a significant expansion of the current test suite and creation of further test suites for other APIs (the number of APIs and requests that can be made is unlimited in the free tier), so it remains a cost-effective option for API testing. As more collaboration between team members (e.g. developers and QA), or significant mocking functionality is required then a paid plan will be necessary.

## REST API
The API tested here is the [JSON Placeholder](https://jsonplaceholder.typicode.com) API, a free fake API for testing and prototyping. The API has six endpoints across a number of different types:
* [/posts](https://jsonplaceholder.typicode.com/posts) - 100 posts
* [/comments](https://jsonplaceholder.typicode.com/comments) - 500 comments
* [/albums](https://jsonplaceholder.typicode.com/albums) - 100 albums
* [/photos](https://jsonplaceholder.typicode.com/photos) - 5000 photos
* [/todos](https://jsonplaceholder.typicode.com/todos) - 200 todos
* [/users](https://jsonplaceholder.typicode.com/users) - 10 users

Some of these resources are related to one another e.g., posts have comments, albums have photos, users make posts etc.

The API has no authorisation/authentication applied so that aspect is not tested here.

## Postman Test Suite
The Postman test suite here is made up of two separate files:
* JSON Placeholder.postman_collection.json
* JSON Placeholder Env.postman_environment.json

The first file is the Postman collection - this contains all the individual requests and tests and as such is the main file within the test suite. The second file is the Postman environment - primarily used to enable fast switching between different configurations e.g. test and production environments.

Both files are in JSON format so can be edited in any text editor, but I wouldn't recommend it. I suggest only opening and editing the files via Postman itself to ensure the right formatting of the files.

### Postman Collection
The Postman collection is a JSON structure containing all the API requests and the associated tests. The collection itself is hierarchical with three main components:
* collection
* folders (and sub-folders)
* requests

Each level in this hierarchy supports pre-request scripts and tests. Both are points at which JavaScript code can be entered with pre-request scripts run **before** the request is sent (as the name suggests) and tests run afterwards. We can exploit both of these to help with our tests.

### Postman Environment
As stated above, environment files are a quick way of switching between different configurations such as test and production. They are also handy for storing variables (see below). These may be variables that may vary between environments (e.g. the base API URL) or 'temporary' variables that are declared in pre-request scripts, with the tests then reading those variables for use in assertions.

### Variables
Postman collections and environments support the declaration and storage of variables. These are available to all requests and any code within the collection, but are not accessible outside the collection. In this case we have six variables defined at the collection level - one per endpoint:
* `postsEndpoint` = `/posts`
* `commentsEndpoint` = `/comments`
* `albumsEndpoint` = `/albums`
* `photosEndpoint` = `/photos`
* `todosEndpoint` = `/todos`
* `usersEndpoint` = `/users`

There are also two 'permanent' variables defined in the environment file:
* `url` = `https://jsonplaceholder.typicode.com`
* `responseTimeout` = `1000`

These variables may be combined, e.g., the URI for a request can be set to `{{url}}{{postsEndpoint}}` rather than `https://jsonplaceholder.typicode.com/posts`. That way if the base URL or endpoint changes we don't need to go through all the requests changing the URI and can just change the value of the variable.

### Structure
The Postman collection is broken down into separate folders per endpoint (each folder is named after the endpoint), although only the Posts endpoint folder has been created at this stage to keep the project light as it is intended to be a template rather than a full implementation of the API testing. Each endpoint folder is then split into separate Positive Tests and Negative Tests folders, the purpose of which is hopefully obvious from the names. Those folders are then split down by response status code (e.g. 200 OK, 404 Not Found) with requests created in these response code folders based on the expected behaviour.

As an example, a GET request to the `/posts` endpoint is expected to return a 200 OK status code as part of the response. This is a positive test of the API i.e. checking it behaves as expected and 200 is a status code associated with a correct response. This request would be found in the following location:
* JSON Placeholder (collection)
    * Posts (folder)
        * Positive Tests (folder)
            * 200 OK (folder)
                * Get All Posts (request)
    
In some cases it is necessary to introduce further structure to the project with sub-folders added below the status code folders. This may be required when the schema of the responses may differ even though the status code is the same e.g. separating responses that return an array of results from those that return a single response. This is all to simplify the actual test code.

There is also a Workflows folder at the top level of the collection, containing sub-folders of daisy-chained requests for test cases that require more than one request to be sent.

### Tests
The Postman collection structure may look unnecessarily complicated but is an essential part of the testing. The aim is to minimise the amount of test code that needs to be written by defining tests at the relevant folder level rather than duplicating the code across all requests within that folder. This way we maintain good software development and coding principles such as DRY (Don't Repeat Yourself) and make it easier to maintain the test suite going forward, especially if changes are required.

For each request we typically want to run the following tests (at least):
* response is received in a timely manner
* response has the right status code 
* response is in the correct format (e.g JSON)
* response schema is correct
* response body contents are correct

Rather than add each test to each request we use the project structure, defining the tests per folder and making use of pre-request scripts to create additional variables to support those tests. Let's examine each layer of the hierarchy in turn.

**JSON Placeholder** (collection)

There is no pre-request script.

The response time test is created here at the collection level and thus applied to all requests within the collection. The maximum expected response time is obtained from an environment variable, and an assertion checks the actual response time is less than that maximum. If we want to change this test we can change the code here, and it will affect all requests in the collection.

**Posts** (folder)

There is no pre-request script.

This is where we test the response is in JSON format. This is done by checking that the `content-type` header is present and that is it set to `application/json` as well as using an in-built assertion that the response is in JSON format. It's a belt and braces check, but it given the minimal additional overhead it is worth doing.

The equivalent folders for the other endpoints (comments, photos etc) would do the same. If the response for all endpoints was expected to be JSON this test could be moved to the parent collection rather than duplicating it across all endpoint folders.

**Positive Tests** (folder)

There is no pre-request script on the Positive Tests.

The tests on this folder use a neat little trick made possible by the project structure. There are two tests: checking the JSON schema is correct and also checking the response body contents are correct. How do we know what the correct schema and expected body are? They are defined lower in the hierarchy via pre-request scripts so by the time these tests are executed (after the request is made) the expected schema and response body are defined.

The Negative Tests folder is similarly configured but there is no schema validation test. Also, as all negative requests are expected to return an empty JSON structure in the body, the response body contents test is simpler and uses a hardcoded empty JSON structure as the expected result.  

**200 OK** (folder)

There is no pre-request script.

Here we test the response status code is as expected - 200 in this case. Sibling folders would have the same structure but with a different expected result e.g. the 404 Not Found folder has a single test asserting that the response has a status code of 404.

**All Posts** (folder)

The pre-request script defines the JSON schema for a response containing an array of posts and writes this to a `postsSchema` variable in the environment file. Thus, before the request is sent `postsSchema` is defined and can then be used in the schema validation test (declared in the Positive Tests folder).

This folder contains a test to verify the results are returned in an array. Again, this is a bit belt & braces given we have verified the JSON schema is correct (and that is an array of posts) but here we also assert that the number of posts in that array is greater than 0. There is also a series of additional tests that run if the response body contents are not as expected in order to make tracking down the differences easier; rather than just being told the response body doesn't match the expected these tests will highlight where the differences are. This is within this folder rather than within the Positive Tests folder because we need slightly different code for an array of posts rather than a single post.

The Single Post folder is very similar to the All Posts folder but with a different schema defined in the pre-request script and different tests to highlight any differences between the actual and expected response bodies.

**Get All Posts** (request)

The expected response body is declared as an environment file variable in the pre-request script. This is most easily obtained by sending the request once manually and copying the response body into here (after first verifying the body is correct manually).

There are no tests on the request.

Other requests are similar but with different expected response bodies declared in the pre-request script.

#### Workflow Tests
In this example project only a single workflow test has been implemented - demonstrating an equivalence between the Posts and Comments endpoints. One can obtain the comments for a given post ID via the Posts endpoint by appending `/comments` to the request URL. One can also use the post ID as a query parameter in a request to the Comments endpoint. Both requests should return the same response (for the same post ID), which is what this workflow test shows. Let's look at how this is structured.

**Workflow Tests** (folder)

This is just a structural folder with no pre-request script or tests.

**Posts & Comments Endpoint Equivalence** (folder)

The pre-request script defines the JSON schema for a response containing an array of comments and writes this to a `commentsSchema` environment variable.

All the tests for this workflow are defined at this folder level. We verify the response has a status code of 200, that the response is in JSON format, the schema is correct, the results are an array (with at least 1 element) and finally that the response body contents are as expected.

**Get Post 1 Comments (Post Endpoint)** (request)

The expected response is declared in the pre-request script and is stored in a variable in the environment file. 

There are no tests for the request but there is a single line of code to set the next request (equivalent to a 'go to'), which is how we ensure these requests are daisy-chained.

**Get Post 1 Comments (Comments Endpoint)** (request)

There is no pre-request script and no tests in this request.

This request runs the tests defined at the folder level above, using the expected response declared in the previous request. This way we show that both requests in this workflow folder are equivalent.

### Pros & Cons
**Pros**
* Good software engineering principles are adopted (e.g. DRY)
* The amount of test code that needs to be written is minimised
* Future changes need to be made in one place only (easier maintenance)
* New requests automatically inherit tests from the folder to which they are added making it faster & easier to extend the project
* A consistent testing approach across all requests
* Consistent test names across requests makes test results easier to review

**Cons**
* It is potentially harder for someone new to the project to understand the Postman collection
* Adding new requests can be more difficult unless you understand the collection structure

### Running the tests
The tests for each request will automatically be run when the corresponding request is sent. The tests can be run in the standard manner within Postman, i.e.
* run the entire collection
* run all tests in an individual folder
* send an individual request

If the JSON Placeholder Env environment file is not selected the requests will fail to send (as the URI will be undefined).

The [Newman](https://learning.postman.com/docs/running-collections/using-newman-cli/command-line-integration-with-newman) command line runner can also be used.

## CI Pipeline
This project contains an implementation of a CI pipeline using [GitHub actions](https://github.com/features/actions). Any push to the `master` branch or any pull request on the `master` branch will trigger the pipeline, which runs in a Linux VM on the cloud within GitHub. The pipeline:
* checks out the repo
* sets up Node v12
* installs npm
* installs Newman (via npm)
* runs the Postman collection via Newman (using the specified environment file)

At the end of the steps the environment tears itself down and produces a [status report](https://github.com/mathare/api-testing-postman/actions)


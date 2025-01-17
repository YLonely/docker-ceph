# Testing

This document contains the Moby code testing guidelines. It should answer any 
questions you may have as an aspiring Moby contributor.

## Test suites

Moby has two test suites (and one legacy test suite):

* Unit tests - use standard `go test` and
  [gotest.tools/assert](https://godoc.org/gotest.tools/assert) assertions. They are located in
  the package they test. Unit tests should be fast and test only their own 
  package.
* API integration tests - use standard `go test` and
  [gotest.tools/assert](https://godoc.org/gotest.tools/assert) assertions. They are located in
  `./integration/<component>` directories, where `component` is: container,
  image, volume, etc. These tests perform HTTP requests to an API endpoint and
  check the HTTP response and daemon state after the call.

The legacy test suite `integration-cli/` is deprecated. No new tests will be 
added to this suite. Any tests in this suite which require updates should be 
ported to either the unit test suite or the new API integration test suite.

## Writing new tests

Most code changes will fall into one of the following categories.

### Writing tests for new features

New code should be covered by unit tests. If the code is difficult to test with
a unit tests then that is a good sign that it should be refactored to make it
easier to reuse and maintain. Consider accepting unexported interfaces instead
of structs so that fakes can be provided for dependencies.

If the new feature includes a completely new API endpoint then a new API 
integration test should be added to cover the success case of that endpoint.

If the new feature does not include a completely new API endpoint consider 
adding the new API fields to the existing test for that endpoint. A new 
integration test should **not** be added for every new API field or API error 
case. Error cases should be handled by unit tests.

### Writing tests for bug fixes

Bugs fixes should include a unit test case which exercises the bug.

A bug fix may also include new assertions in an existing integration tests for the
API endpoint.

### Integration tests environment considerations

When adding new tests or modifying existing test under `integration/`, testing 
environment should be properly considered. `skip.If` from 
[gotest.tools/skip](https://godoc.org/gotest.tools/skip) can be used to make the 
test run conditionally. Full testing environment conditions can be found at 
[environment.go](https://github.com/moby/moby/blob/cb37987ee11655ed6bbef663d245e55922354c68/internal/test/environment/environment.go)

Here is a quick example. If the test needs to interact with a docker daemon on 
the same host, the following condition should be checked within the test code

```go
skip.If(t, testEnv.IsRemoteDaemon())
// your integration test code
```

If a remote daemon is detected, the test will be skipped.

## Running tests

To run the unit test suite:

```
make test-unit
```

or `hack/test/unit` from inside a `BINDDIR=. make shell` container or properly
configured environment.

The following environment variables may be used to run a subset of tests:

* `TESTDIRS` - paths to directories to be tested, defaults to `./...`
* `TESTFLAGS` - flags passed to `go test`, to run tests which match a pattern
  use `TESTFLAGS="-test.run TestNameOrPrefix"`

To run the integration test suite:

```
make test-integration
```

You can change a version of golang used for building stuff that is being tested
by setting `GO_VERSION` variable, for example:

```
make GO_VERSION=1.12.8 test
```

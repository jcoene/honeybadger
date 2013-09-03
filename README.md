# Honeybadger Go Client

This is an unofficial notifier library for integrating Go applications with [Honeybadger](http://honeybadger.io).

## Usage

First you'll need to import the library and set your Honeybadger API key and an environment name for your application:

```go
import "github.com/jcoene/honeybadger"

func main() {
  // Set the API key
  honeybadger.ApiKey = "abcdef"
  // Set the application environment
  honeybadger.Environment = "production"
}
```

Later (probably when recovering from some kind of panic or error), you can create and send an error report:

```go
func DoStuff() (err error) {
  if err = doOtherThing(); err != nil {
    // Create a new report with 0 call stack inflation (more on that later)
    // Give it the error we received as the message (could be anything)
    report, err2 := honeybadger.NewReport(0, err)

    // Send the error (asynchronously in a Goroutine)
    report.Dispatch()
  }
}

It's also possible (advisable) to add some context for the failure:

```go
// Create the report
report, _ := honeybadger.NewReport(0, err)

// Set the request URL
report.Request.URL = myHttpReq.URL

// Set all of the incoming request headers. Could be anything.
for k, v := range myHttpReq.Header {
  report.AddContext(k, v[0])
}
```

Errors are automatically labeled and given backtraces based on the call stack. The automatic labels only work if the Honeybadger library can properly determine the origin of the error. You'll need to supply the depth of stack inflation as the first argument to NewReport (0 assumes that honeybadger.NewReport is called directly from the function reporting the error, 1 being one call removed in case of the use of a helper function).

For example, let's say you want to have a helper function in your service to report errors, it should call NewReport with a depth of 1:

```go
  func Get(req, resp, ...) {
    if err = doOtherThing; err != nil {
      // Oh no, let ops know that things aren't going so well!
      reportError(req, resp, err)
    }
  }

  ...

  func reportError(req, resp, err) {
    // Create the report with an inflated stack depth of 1
    report, err2 := honeybadger.NewReport(1, err)

    // Fill in a bunch of useful information from the request
    report.Request.URL = req.URL
    ...

    // Send
    report.Dispatch()
  }
```

## License

MIT

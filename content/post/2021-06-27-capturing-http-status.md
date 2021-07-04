+++
tags = ["go", "golang"]
categories = []
title = "Capturing HTTP status in Go. Or, it's about compostion"
date = 2021-06-27T09:29:18+01:00
draft = false
+++

Rob Pike [says it best](https://commandcenter.blogspot.com/2012/06/less-is-exponentially-more.html):
> If C++ and Java are about type hierarchies and the taxonomy of types,
> Go is about composition.

Let's illustrate this with an example from `net/http`, a fantastic package.

Say you've built a server in Go, and you would like to know when handlers
return a 200-range code or a 500-range code.
For example, you may want to capture metrics to collect in your *Prometheus*. Or,
you may want to alert your crash-reporting system on every 500.

However, the canonical Go signature for HTTP handlers is:

``` go
func myHandler(http.ResponseWriter, *http.Request)
```

The function returns nothing. How can you capture the HTTP status after execution?

If you do a search on the web, you'll very easily find the following approach:
`http.ResponseWriter` is an interface, so you can create an implementation of
your own, with a field to hold the returned HTTP status.

Say

``` go
type myWriter {
    w http.ResponseWriter
    Status int
}
```

All you need to do is implement the three methods in the `http.ResponseWriter`
interface, and add a bit extra to `WriteHeader`:

``` go
func (m *myResponse) WriteHeader(statusCode int) {
    // m.w.WriteHeader(statusCode)
    m.Status = statusCode
}
```

After a `myResponse` has been written, you can read its Status. How are you
supposed to ensure that the existing handlers in your codebase use `myResponse`?

Again, thanks to the `http.ResponseWriter` being an interface, and a small one,
it is easy to write a *middleware* to wrap existing handlers so they use the
custom response writer.

``` go
func wrapHandler(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        mw := &myResponse{
            w: w,
        }
        next(mw, r)
        log.Println("status was", mw.Status)
    }
}
```


``` go
// AppCoords represents the data we care about w.r.t. MSA metrics
type AppCoords struct {
    Name    string
    Country string
    Peril   string
    Model   string
}

// AppHandler represents a web app handler, only returning error
// i.e. it requires transformation to serve as a http.Handler
type AppHandler interface {
    ServeApp(http.ResponseWriter, *http.Request) (AppCoords, int, error)
}

func WrapAppHandler(counter *prometheus.CounterVec, lg *log.Logger,
    ah AppHandler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {

        coords, statusCode, err := ah.ServeApp(w, r)
```

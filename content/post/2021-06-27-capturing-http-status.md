+++
tags = ["go", "golang", "coding"]
categories = []
title = "HTTP status in Go. Or, composition rules"
date = 2021-06-27T09:29:18+01:00
draft = false
+++

> If C++ and Java are about type hierarchies and the taxonomy of types,
> Go is about composition.

from Rob Pike in [*Less is exponentially more.*](https://commandcenter.blogspot.com/2012/06/less-is-exponentially-more.html)

We can see Go's pervasive philosophy of composition at play in the
[`net/http`](https://golang.org/pkg/net/http/) package.

Say you've built a server in Go, and you would like to know when handlers
return a 200-range code or a 500-range code.
For example, you may want to capture metrics to for your *Prometheus*. Or,
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
interface, and you'll be able to use it in handlers.
You add a bit extra to `WriteHeader`:

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
It is convenient that the `http.HandlerFunc` interface encodes the signature of
HTTP handlers.

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

If you've had the foresight to package your handlers across your codebase so that
the URL and the handler function are both available, it will be easy to wrap
all your endpoints with the `wrapHandler` middleware.

``` go
type EndPoint struct {
    Pattern string
    Handler http.HandlerFunc
}

    …
    for _, ep := range myFooService.EndPoints {
        myMux.HandleFunc(ep.Pattern, wrapHandler(ep.Handler))
    }
    …
```

Note:

* your custom `ResponseWriter` is transparent to the rest of the codebase. Your
  colleagues do not need to make any changes to the HTTP handlers they wrote
* your middleware is also transparent to the existing HTTP handlers. You just
  added it like a LEGO piece. Someone else could write an additional
  middleware to add behavior, without modifying yours

Go makes this type of composition very easy. But it's not just that the language
offers interfaces and first-class functions. It's a whole philosophy of use,
one that is clearly informed by the UNIX design of pipes, and its continuation
in *Plan 9*.[^1]

In a previous job, I was creating applications for the Catastrophe Model sector
(an engineering branch of the insurance industry). I had started a
Prometheus service to capture internal usage metrics.

I only had a few endpoints I wanted tracked, which each generated a different
kind of report. All reports were highly dependent on country,
model (commercial *catastrophe model* used),
and peril (earthquake, hurricane, flood, …)

My approach then was to reify the bits I was interested in tracking, which were
more than just an HTTP code:

``` go
// AppCoords represents the data we care about w.r.t. metrics
type AppCoords struct {
    ReportKind string
    Country    string
    Peril      string
    Model      string
}

// AppHandler represents a web app handler, only returning error
// i.e. it requires transformation to serve as a http.Handler
type AppHandler interface {
    ServeApp(http.Re[…], *http.Request) (AppCoords, int, error)
}

func WrapAppHandler(counter *prometheus.CounterVec,
    ah AppHandler) http.Handler {

    return http.HandlerFunc(func(w http.Re[…], r *http.Request) {

        coords, statusCode, err := ah.ServeApp(w, r)
        // my prometheus code followed here
```

This is a very different approach, and today I might have done things differently.
The point is that hooking metrics up was easy to do, and accommodated different
paths.

Because the *interface* abstraction in Go is so pervasive and flexible, you can
think of myriad ways to mix and match, reuse, and build new small pieces that
fit together.

But in order for this to work well, interfaces should be small.
An interface with 23 methods is hardly an invitation for reuse and composition.

Again, Rob Pike says it best, in [Go Proverbs](https://go-proverbs.github.io)
[(video)](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=317s)
> The bigger the interface, the weaker the abstraction.

[^1]: It's no coincidence, given that Rob Pike and Ken Thompson, as well as other
Go core team members, worked on both UNIX and Plan 9.

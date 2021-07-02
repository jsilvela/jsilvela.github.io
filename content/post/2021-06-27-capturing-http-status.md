+++
tags = ["go", "golang"]
categories = []
title = "Capturing HTTP status from Handlers in Go"
date = 2021-08-27T09:29:18+01:00
draft = true
+++

Say you would like to know whether a Go HTTP handler returned a 200-range code
or a 500-range code, after finishing.
For example, you may want to capture metrics to collect in your Prometheus. Or,
you may want to alert your crash-reporting system on every 500.

However, the canonical Go signature for HTTP handlers is:

``` go
func myHandler(http.ResponseWriter, *http.Request)
```

There is no return code in the function. How can you capture the HTTP status?

I learned this trick back in 2017, don't remember exactly from where. I was
looking for it now, and it doesn't seem to appear in Stack overflow articles
or on independent posts on Go, and it's a shame.

So, the trick:

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

# appmetrics

This library is moved to [go-collection](https://github.com/teran/go-collection)
repository for unified experience and simplifying maintenance process.
This repository will **not** be deleted for backward compatibility.

[![Verify](https://github.com/teran/appmetrics/actions/workflows/verify.yml/badge.svg?branch=master)](https://github.com/teran/appmetrics/actions/workflows/verify.yml)
[![Go Report Card](https://goreportcard.com/badge/github.com/teran/appmetrics)](https://goreportcard.com/report/github.com/teran/appmetrics)
[![Go Reference](https://pkg.go.dev/badge/github.com/teran/appmetrics.svg)](https://pkg.go.dev/github.com/teran/appmetrics)

Easy-to-use library to create metrics &amp; probes handlers

## Example

```go
package main

import (
    "github.com/labstack/echo-contrib/echoprometheus"
    echo "github.com/labstack/echo/v4"
    "github.com/labstack/echo/v4/middleware"
    "github.com/teran/appmetrics"
    "golang.org/x/sync/errgroup"
)

func main() {
    g, _ := errgroup.WithContext(context.Background())

    // Initialize labstack/echo for further use to serve metrics
    me := echo.New()
    me.Use(middleware.Logger())
    me.Use(echoprometheus.NewMiddleware("testapp"))
    me.Use(middleware.Recover())

    checkFn := func() error {
        // check function contains actual checks for the app
        // so it could contain db.Ping() and other checks
        return nil
    }

    // New accepts 3 functions to check on different stages:
    // livenessProbeFn, readinessProbeFn, startupProbeFn
    //
    metrics := appmetrics.New(checkFn, checkFn, checkFn)

    // Register HTTP handlers
    metrics.Register(me)

    // Run HTTP server in goroutine
    g.Go(func() error {
        srv := http.Server{
            Addr:    ":8081",
            Handler: me,
        }

        return srv.ListenAndServe()
    })

    // Lock current goroutine
    if err := g.Wait(); err != nil {
        panic(err)
    }
}
```

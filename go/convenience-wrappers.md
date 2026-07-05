## Convenience Wrappers

When you write a package that exposes funcs with various parameters, it is
always a good idea to also write a thin wrapper over it, with the common
default paramteres.

A simple API for common cases, and a configurable API for advaced cases.

Example: 
```go
password.Generate()   // uses GenerateN() with a default value
password.GenerateN(n) // configuratble func

// Standard Library Examples:
http.Get()        // uses NewRequest() in the background
http.Post()       // same here
http.NewRequest() // the base configurable function
```

```go
func (c *Client) Get(url strig) (resp.Response, err error) {
    req, err := NewRequst("GET", url, nil)
    if err != nil {
        return nil, err
    }
    return c.Do(req)
}
```

Here, if you wanted, you could directly use `NewRequest()` to build the `Get` functionality, but because it is known that `NewRequest()` is usually used for GET requests, the package exposes `Get()`...

*Most Go standard packages use this pattern*
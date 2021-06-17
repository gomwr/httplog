httplog
=======

This is a simple forked of [go-chi/httplog](https://github.com/go-chi/httplog) middleware, with additional support of dumping request and response body. This is mostly useful for auditing purpose.

## Differences

Changes made in this repo only takes effect in non-concise mode. Otherwise, the behavior would remain the same.
* response body limit is now configurable with default changed from 512 to 128000 bytes
* In non-concise mode
  * instead of duplicating httpRequest log between the context and message, header and body is logged under  httpRequestExtra
  * response body is always included in the log message (go-chi/httplog include response only when http code is 4xx)

## Log Example

Making POST request.
```
curl --request POST \
  --url http://localhost:5555/reflect \
  --header 'content-type: application/json' \
  --data '{"title": "example"}'
```

Log output.
```json
{
    "level": "info",
    "service": "httplog-example",
    "httpRequest": {
        "proto": "HTTP/1.1",
        "remoteIP": "[::1]:50389",
        "requestID": "machine.local/8Ecrdfj1sf-000007",
        "requestMethod": "POST",
        "requestPath": "/reflect",
        "requestURL": "http://localhost:5555/reflect"
    },
    "httpRequestExtra": {
        "body": "{\"title\": \"example\"}",
        "header": {
            "accept": "*/*",
            "content-length": "20",
            "content-type": "application/json",
            "user-agent": "insomnia/2020.4.1"
        }
    },
    "timestamp": "2021-02-19T23:38:24.840108+07:00",
    "message": "Request: POST /reflect"
}
{
    "level": "info",
    "service": "httplog-example",
    "httpRequest": {
        "proto": "HTTP/1.1",
        "remoteIP": "[::1]:50389",
        "requestID": "machine.local/8Ecrdfj1sf-000007",
        "requestMethod": "POST",
        "requestPath": "/reflect",
        "requestURL": "http://localhost:5555/reflect"
    },
    "httpResponse": {
        "body": "POST /reflect HTTP/1.1\r\nHost: localhost:5555\r\nAccept: */*\r\nContent-Length: 20\r\nContent-Type: application/json\r\nUser-Agent: insomnia/2020.4.1\r\n\r\n{\"title\": \"example\"}",
        "bytes": 164,
        "elapsed": 0.026024,
        "status": 200
    },
    "timestamp": "2021-02-19T23:38:24.840224+07:00",
    "message": "Response: 200 OK"
}
```

## Server Example

Also available in [_example/](./_example/)

```golang
package main

import (
  "net/http"
  "net/http/httputil"

  "github.com/go-chi/chi/v5"
  "github.com/go-chi/chi/v5/middleware"

  "github.com/manat/httplog"
)

func main() {
	logger := httplog.NewLogger("httplog-example", httplog.Options{
		JSON: true,
	})

	// Service
	r := chi.NewRouter()
	r.Use(httplog.RequestLogger(logger))

	r.Get("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("hello world"))
	})

	r.Post("/reflect", func(w http.ResponseWriter, r *http.Request) {
		dump, _ := httputil.DumpRequest(r, true)
		w.Write(dump)
	})

	http.ListenAndServe(":5555", r)
}
```

## License

MIT

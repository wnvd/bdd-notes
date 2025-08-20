# https server in go

## What is a Server?

A web server is just a computer that serves data over a network, typically
the internet. Servers run software that listens for incoming request
from clients. When a request is received, the server responds with the
requested data.

In go, we use a new goroutine for each request to handle them concurrently.

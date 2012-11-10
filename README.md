HTTP Authentication implementation in Go
========================================

This is an implementation of HTTP Basic and HTTP Digest authentication
in Go language. It is designed as a simple wrapper for
http.RequestHandler functions.

Features
--------
 
 * Supports HTTP Basic and HTTP Digest authentication.
 * Supports htpasswd and htdigest formatted files.
 * Automatic reloading of password files.
 * Pluggable interface for user/password storage.
 * Supports MD5 and SHA1 for Basic authentication password storage.
 * Configurable Digest nonce cache size with expiration.
 
Example usage
-------------

This is a complete working example for Basic auth:

    package main

    import (
        "fmt"
        "http"
        auth "github.com/abbot/go-http-auth"
    )

    func Secret(user, realm string) string {
  	    if user == "john" {
  		    // password is "hello"
  		    return "$1$dlPL2MqE$oQmn16q49SqdmhenQuNgs1"
        }
	    return ""
    }

    func handle(w http.ResponseWriter, r *auth.AuthenticatedRequest) {
	    fmt.Fprintf(w, "<html><body><h1>Hello, %s!</h1></body></html>", r.Username)
    }

    func main() {
	    authenticator := auth.BasicAuthenticator("example.com", Secret)
	    http.HandleFunc("/", authenticator(handle))
	    http.ListenAndServe(":8080", nil)
    }

This is a complete working example for Digest auth:

    package main

    import (
        "fmt"
        "http"
        auth "github.com/abbot/go-http-auth"
    )

    func Secret(user, realm string) string {
  	    if user == "john" {
  		    // password is "hello"
  		    return "b98e16cbc3d01734b264adba7baa3bf9"
        }
	    return ""
    }

    func handle(w http.ResponseWriter, r *auth.AuthenticatedRequest) {
	    fmt.Fprintf(w, "<html><body><h1>Hello, %s!</h1></body></html>", r.Username)
    }

    func main() {
	    authenticator := auth.DigestAuthenticator("example.com", Secret)
	    http.HandleFunc("/", authenticator(handle))
	    http.ListenAndServe(":8080", nil)
    }


The following example demonstrates using Digest auth with the secrets loaded from a file and with the Auth interface.
First create a digest file with `htdigest -c test.htdigest example.com john`, then run this from the same directory:

    package main

    import (
        "fmt"
        "net/http"
        auth "github.com/kig/go-http-auth"
    )

    func main() {
	    da := auth.MakeDigestAuth("example.com", auth.HtdigestFileProvider("test.htdigest"))
	    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
                    user, _ := da.CheckAuth(r)
                    if user == "" {
                            da.RequireAuth(w, r)
                    } else {
                            fmt.Fprintf(w, "<html><body><h1>Hello, %s!</h1></body></html>", user)
                    }
            })
	    http.ListenAndServe(":8080", nil)
    }

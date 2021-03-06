# Slimane

<img src="https://raw.githubusercontent.com/noppoMan/Slimane/master/logo/Slimane_logo.jpg" width=250>

## OverView
Slimane is an express inspired web framework for Swift that works on OSX and Ubuntu.


- [x] 100% Asynchronous
- [x] Unopinionated and Minimalist
- [x] Adopts [Open Swift](https://github.com/open-swift)

### Programming Guid
Getting ready

## Slimane Project Page 🎉
Various types of libraries are available from here.

https://github.com/slimane-swift

## Community
The entire Slimane code base is licensed under MIT. By contributing to Slimane you are contributing to an open and engaged community of brilliant Swift programmers. Join us on [Slack](https://slimane-swift-slackin.herokuapp.com/) to get to know us!

## Getting Started

### Install Guide
[Here is an install guides for each operating systems](https://github.com/noppoMan/Slimane/wiki/Install-Guide)

### Documentation
[Here is a Documentation for Slimane.](https://github.com/noppoMan/Slimane/wiki)

## Usage

Starting the application takes slight lines.

```swift
import Slimane

let app = Slimane()

app.get("/") { req, responder in
    responder {
        Response(body: "Welcome Slimane!")
    }
}

try! app.listen()
```

### Generate a new Slimane App via slimane-cli(Requires Node.js v4 or later)
```sh
npm i -g slimane-cli
```

```sh
slimane new YourAppName
cd YourAppName
slimane build
slimane run
```

**That's it!**

## Routing

```swift
app.get("/articles/:id") { req, responder in
    responder {
        Response(body: "Article ID is: \(req.params["id"]!)")
    }
}
```

#### Methods
* get
* options
* post
* put
* patch
* delete


## Middlewares
Middleware is functions that have access to the http request, the http response, and the next function in the application' s request-response cycle.

### Handy

```swift
app.use { req, next, result in
    print("[\(Suv.Time())] \(req.uri.path ?? "/")")
    next.respond(to: req, result: result)
}
```

### AsyncMiddleware

```swift
struct FooMiddleware: AsyncMiddleware {
    func respond(to request: Request, chainingTo next: AsyncResponder, result: (Void throws -> Response) -> Void) {
        do {
            var request = request
            let foo = try throwableFoo()
            request.foo = foo
            next.respond(to: request, result: result) // Chain the next middleware
        } catch {
            // Respond to the content immediately.
            result {
                Response(status: .internalServerError, body: "\(error)")
            }
        }
    }
}

app.use(FooMiddleware())
```

## Request/Response

We are using S4.Request and S4.Response
See more detail, please visit https://github.com/open-swift/S4

## Static Files/Assets

Just register the `Slimane.Static()` into middleware chains

```swift
app.use(Slimane.Static(root: "/path/to/your/public"))
```

## Cookie

#### request.cookies: `Set<Cookie>`

request.cookies is Readonly.
```swift
req.cookies["session-id"]
```

**Cookie** is declared in HTTP.Cookie. See more to visit https://github.com/slimane-swift/HTTP

#### response.cookies: `Set<AttributedCookie>`

response.cookies is Writable.

```swift
let setCookie = AttributedCookie(....)
res.cookies = Set<setCookie>
```
**AttributedCookie** is declared in HTTP.AttributedCookie. See more to visit https://github.com/slimane-swift/HTTP


## Session

Register SessionMiddleware into the middleware chains.
See more detail for SessionMiddleware to visit https://github.com/slimane-swift/SessionMiddleware

```swift
import Slimane
import SessionMiddleware

let app = Slimane()

// SessionConfig
let sesConf = SessionConfig(
    secret: "my-secret-value",
    expires: 180,
    HTTPOnly: true
)

// Enable to use session in Slimane
app.use(SessionMiddleware(conf: sesConf))

app.get("/") { req, responder
    // set data into the session
    req.session["foo"] = "bar"

    req.session.id // show session id

    responder {
        Response()
    }
}
```

### Available Session Stores
* MemoryStore
* SessionRedisStore


## Body Data

Register BodyParser into the middleware chains.

```swift
app.use(BodyParser())
```

#### request.json `Zewo.JSON`

Can get parsed json data throgh the req.json when content-type is `application/json`

```swift
req.json?["foo"]
```

#### request.formData `[String: String]`

Can get parsed form data throgh the req.formData when content-type is `application/x-www-form-urlencoded`

```swift
req.formData?["foo"]
```

## Views/Template Engines
* Add the [Render](https://github.com/slimane-swift/Render) module into the Package.swift
* Add the [MustacheViewEngine](https://github.com/slimane-swift/MustacheViewEngine) module into the Package.swift

Then, You can use render function in Slimane. and pass the render object to the `custom` label for Response initializer.

```swift
app.get("/render") { req, responder in
    responder {
        let render = Render(engine: MustacheViewEngine(templateData: ["foo": "bar"]), path "index")
        Response(custom: render)
    }
}
```

## Create your own ViewEngine

Getting ready

## Working with Cluster

A single instance of Slimane runs in a single thread. To take advantage of multi-core systems the user will sometimes want to launch a cluster of Slimane processes to handle the load.


Here is an easy example for working with Suv.Cluster

```swift
// For Cluster app
if Cluster.isMaster {
    for _ in 0..<OS.cpus().count {
        let worker = try! Cluster.fork(silent: false)
    }

    try! Slimane().listen()
} else {
    let app = Slimane()
    app.get("/") { req, responder in
        responder {
            Response(body: "Hello! I'm \(CommandLine.pid)")
        }
    }

    try! app.listen()
}
```

## IPC between Master and Worker Processes

Inter process message between master and workers

### On Master
```swift
var worker = try! Cluster.fork(silent: false)

// Send message to the worker
worker.send(.Message("message from master"))

// Receive event from the worker
worker.onIPC { event in
    if case .message(let str) = event {
        print(str)
    }

    else if case .online = event {
        print("Worker: \(worker.id) is online")
    }

    else if case .exit(let status) = event {
        print("Worker: \(worker.id) is dead. status: \(status)")
        let worker = try! Cluster.fork(silent: false)
        observeWorker(worker)
    }
}
```

### On Worker
```swift

// Receive event from the master
Process.onIPC { event in
    if case .message(let str) = event {
        print(str)
    }
}

// Send message to the master
Process.send(.message("Hey!"))
```

## Respond to the Streaming Content

You can respond to the streaming content with `Body.asyncSender`
Here is an example for websocket response with [WS](https://github.com/slimane-swift/WS)

```swift
import WS
import Slimane

let app = Slimane()

let wsServer = WebSocketServer { socket, request in
    socket.onText { text in
        print(text)
    }
}

app.use(wsServer)

app.get("/") { req, responder in
    responder {
        Response(body: "html text here....")
    }
}

try! app.listen()
```

## Extras

### Working with blocking functions
We have `Process.qwork` to run blocking functions in a separated thread.

It allows potentially any third-party libraries to be used with the event-loop paradigm.

```swift
let onThread = { ctx in
    do 
        ctx.storage["result"] = try blokingOperation()
    } catch {
        ctx.storage["error"] = error
    }
}

let onFinish = { ctx in
    if let error = ctx.storage["error"] as? Error {
        print(error)
        return
    }
    
    print(ctx.storage["result"])
}

Process.qwork(onThread: onThread, onFinish: onFinish)
```

### Promise
We have [Thrush](https://github.com/noppoMan/Thrush) to use Promise apis in the app to make beautiful asynchronous flow.
Thrush has similar apis to the [ES promise](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise).
For more detail, visit https://github.com/noppoMan/Thrush

Here is a replacement codes of the [Working with blocking functions](#working-with-blocking-functions) section with `Promise`

```swift
import Thrush

extension DB {
    func execute(sql: String) -> Promise<FooResult> {
        return Promise<FooResult> { resolve, reject in
            let onThread = { ctx in
                do 
                    ctx.storage["result"] = try blockingSqlQuery(sql)
                } catch {
                    ctx.storage["error"] = error
                }
            }
            
            let onFinish = { ctx in
                if let error = ctx.storage["error"] as? Error {
                    reject(error)
                    return
                }
                
                resolve(ctx.storage["result"] as! FooResult)
            }
            
            Process.qwork(onThread: onThread, onFinish: onFinish)
        }
    }
}

let db = DB(host: "localhost")

db.execute("insert into users (id, name) values (1, 'jack')").then {
    print($0)
}
.`catch` {
    print($0)
}
.finally {
    print("Done")
}
```


## Handling Errors

Easy to override Default Error Handler with replace `app.errorHandler` to your costume handler.
All of the errors that occurred in Slimane's lifecycles are passed as first argument of errorHandler even RouteNotFound.

```swift
let app = Slimane()

app.errorHandler = myErrorHandler

func myErrorHandler(error: ErrorProtocol) -> Response {
    let response: Response
    switch error {
        case Costume.invalidPrivilegeError
            response = Response(status: .forbidden, body: "Forbidden")
        case Costume.resourceNotFoundError(let name)
            response = Response(status: .notFound, body: "\(name) is not found")
        default:
            response = Response(status: .badRequest, body: "\(error)")
    }

    return response
}

try! app.listen()
```

## Package.swift

```swift
import PackageDescription

let package = Package(
      name: "MySlimaneApp",
      dependencies: [
          .Package(url: "https://github.com/noppoMan/Slimane.git", majorVersion: 0, minor: 8),
      ]
)
```

## License

Slimane is released under the MIT license. See LICENSE for details.

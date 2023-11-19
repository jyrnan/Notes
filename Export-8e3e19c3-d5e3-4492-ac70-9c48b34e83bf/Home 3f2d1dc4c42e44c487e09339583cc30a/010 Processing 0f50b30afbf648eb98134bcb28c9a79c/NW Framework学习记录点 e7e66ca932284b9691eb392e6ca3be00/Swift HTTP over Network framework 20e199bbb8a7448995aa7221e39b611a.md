# Swift HTTP over Network framework

另自己没有想到的是，这段代码居然可以直接在playground里面运行，可以直接建立tls连接，而且我还试了连接google，可以获得正确response😺

```swift
import Network
let host: NWEndpoint.Host            = .init( "httpbin.org" )
let port: NWEndpoint.Port            = .https
let tlsConfig: NWProtocolTLS.Options = .init()
let parameters: NWParameters         = .init( tls: tlsConfig )
let connection: NWConnection         = .init( host: host, port: port, using: parameters )
connection.stateUpdateHandler        = { state in
print( "State Update: \( state )" )
}
connection.start( queue: DispatchQueue.global() )
let method         = "POST"
let uri            = "/"
let httpVersion    = "HTTP/1.1"
let headers        = "Host: \( host )\r\n"
let body           = ""
let rawHTTPRequest = "\( method ) \( uri ) \( httpVersion )\r\n\( headers )\r\n\( body )"
connection.send( content: rawHTTPRequest.data( using: .ascii ), completion: .contentProcessed( { error in
print( "Processed, error: \( error )" )
    connection.receiveMessage { data, _, completed, error in
if let data = data {
print( "Response: \( String( data: data, encoding: .ascii ) ), completed: \( completed ), error: \( error )" ) // .ascii? utf8
        } else {
print( "Response: nil, completed: \( completed ), error: \( error )" )
        }
    }
}))
```

[https://gist.github.com/TyrfingMjolnir/d73c012b9e3110de51d7e5a04ab3b307](https://gist.github.com/TyrfingMjolnir/d73c012b9e3110de51d7e5a04ab3b307)
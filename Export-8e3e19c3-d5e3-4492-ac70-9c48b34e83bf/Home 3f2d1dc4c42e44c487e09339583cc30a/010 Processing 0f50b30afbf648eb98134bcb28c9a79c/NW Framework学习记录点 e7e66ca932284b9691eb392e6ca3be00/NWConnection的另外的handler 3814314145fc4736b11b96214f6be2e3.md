# NWConnection的另外的handler

```swift
/// Set a block to be called when the connection's viability changes, which may be called
    /// multiple times until the connection is cancelled.
    ///
    /// Connections that are not currently viable do not have a route, and packets will not be
    /// sent or received. There is a possibility that the connection will become viable
    /// again when network connectivity changes.
    final public var viabilityUpdateHandler: ((_ isViable: Bool) -> Void)?

    /// A better path being available indicates that the system thinks there is a preferred path or
    /// interface to use, compared to the one this connection is actively using. As an example, the
    /// connection is established over an expensive cellular interface and an unmetered Wi-Fi interface
    /// is now available.
    ///
    /// Set a block to be called when a better path becomes available or unavailable, which may be called
    /// multiple times until the connection is cancelled.
    ///
    /// When a better path is available, if it is possible to migrate work from this connection to a new connection,
    /// create a new connection to the endpoint. Continue doing work on this connection until the new connection is
    /// ready. Once ready, transition work to the new connection and cancel this one.
    final public var betterPathUpdateHandler: ((_ betterPathAvailable: Bool) -> Void)?
```
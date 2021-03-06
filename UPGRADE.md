# UPGRADE INSTRUCTIONS

## 1.5

* `bt.BaseClientBuilder#runtime(BtRuntime)` is now protected instead of public. Use a factory method `bt.Bt#client(BtRuntime)` to attach the newly created client to a shared runtime.
* Static contribution methods in `bt.module.ServiceModule` and `bt.module.ProtocolModule` have been replaced by _module extenders_, which provide a clearer and more concise API for contributing custom extensions to the core. Instead of invoking individual contributions methods, downstream modules should now call `bt.module.ServiceModule#extend(Binder)` or `bt.module.ProtocolModule#extend(Binder)` and use methods in the returned builder instance, e.g.:

```java
import com.google.inject.Binder;
import com.google.inject.Module;

import bt.module.ProtocolModule;

public class MyModule implements Module {
    
    @Override
    public void configure(Binder binder) {
        ProtocolModule.extend(binder)
            .addMessageHandler(20, ExtendedProtocol.class)
            .addExtendedMessageHandler("ut_metadata", UtMetadataMessageHandler.class);
    }
}
```
* New centralized mechanism was introduced for publishing/receiving events. It's represented by two DI services: `bt.event.EventSink` for publishing and `bt.event.EventSource` for subscriptions. Core services, that previously provided a custom API for subscribing to events, were updated to use the new mechanism, and the old methods and interfaces have been removed, namely:
    - `bt.net.PeerActivityListener` removed
    - `bt.torrent.TorrentSession` does not implement ``bt.net.PeerActivityListener` anymore
    - methods were removed in `bt.peer.IPeerRegistry`:
        - `addPeerConsumer(Torrent torrent, Consumer<Peer> consumer)`
        - `addPeerConsumer(TorrentId torrentId, Consumer<Peer> consumer)`
        - `removePeerConsumers(Torrent torrent)`
        - `removePeerConsumers(TorrentId torrentId)`
    - method was removed in `bt.net.IPeerConnectionPool`:
        - `addConnectionListener(PeerActivityListener listener)`
* `bt.runtime.BtClient#getSession()` now returns a `java.util.Optional`, which is going to be empty until the session is created
* `bt.torrent.TorrentSession#getTorrent()` now returns a `java.util.Optional`, which is going to be empty until the metadata is fetched
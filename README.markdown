# New methods added
I needed to process packets as they were received and I found out from the libpcap documentation that I needed to enable immediate_mode to do this. 
The libpcap documentation says that it is enabled by default in some versions, but I guess I was out of luck.
I had to add this feature, which was not difficult at all thanks to the good code of the package authors.
But luck was not on my side again.
It is possible to enable immediate_mode only if PcapHandle is not activated. But this cannot be achieved using openLive because PcapHandle is created inside this method, then all settings from openLine parameters are applied and PcapHandle is activated.
from libpcap code:
```
p = pcap_create(device, errbuf);
if (p == NULL)
    return (NULL);
status = pcap_set_snaplen(p, snaplen);
if (status < 0)
    goto fail;
status = pcap_set_promisc(p, promisc);
if (status < 0)
    goto fail;
status = pcap_set_timeout(p, to_ms);
if (status < 0)
    goto fail;
p->oldstyle = 1;
status = pcap_activate(p);
if (status < 0)
    goto fail;
return (p);
```
So I had to add all these methods:
 - create
 - activate
 - setSnapLen
 - setPromisc
 - setImmediateMode

except for setTimeout because I'm using immediate_mode which, according to the documentation, doesn't use timeouts.

code example:
 ```
captureOnPort :: [Int] -> IO ()
captureOnPort ports' = do
    countRef <- newIORef 0
    p <- create "enp1s0"
    setDirection p In
    setImmediateMode p True
    setSnapLen p 60
    setPromisc p False
    activate p
    setFilter p (portsStr ports') True 0
    _ <- loop p (-1) (\hdr ptr -> printIt countRef ports' hdr ptr)
    return ()
```
note that filters are applied to the activated PcapHandle.

# A Haskell wrapper around the C libpcap library.

It provides Haskell bindings for most of the libpcap API as of libpcap
version 0.9.7.  The bindings are divided into a very efficient
low-level wrapper, Network.Pcap.Base, and a higher-level module,
Network.Pcap, that's easier to use.

To install:

    cabal install pcap


# Get involved!

Please report bugs via the
[github issue tracker](https://github.org/bos/pcap).

There's also a [git mirror](http://github.com/bos/pcap):

* `git clone git://github.com/bos/pcap.git`

Master [Mercurial repository](http://bitbucket.org/bos/pcap):

* `hg clone http://bitbucket.org/bos/pcap`

(You can create and contribute changes using either Mercurial or git.)


# Authors

This library was originally written by Gregory Wright, with contributions
by Dominic Steinitz.  The current maintainer is Bryan O'Sullivan,
<bos@serpentine.com>.

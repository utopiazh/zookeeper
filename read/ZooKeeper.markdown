# ZooKeeper Code Reading 
## ZooKeeper.java

(1) Request-Response workflow

1. Create a requestHeader, set request type
'''
	RequestHeader h = new RequestHeader();
	h.setType(ZooDefs.OpCode.getData);
'''
2. Create a request 
'''
    GetDataRequest request = new GetDataRequest();
    request.setPath(serverPath);
    request.setWatch(watcher != null);
'''
3. Create a reponse
'''
    GetDataResponse response = new GetDataResponse();
'''
4. Submit request to ClientCnxn
'''
    ReplyHeader r = cnxn.submitRequest(h, request, response, wcb);
    if (r.getErr() != 0) {
        throw KeeperException.create(KeeperException.Code.get(r.getErr()),
                clientPath);
    }
'''
5. Wait for response
'''
    response.getData();
'''

For ZooKeeper synchronous API, ClientCnxn::submitRequest() is blocking:
'''
    public ReplyHeader submitRequest(RequestHeader h, Record request,
            Record response, WatchRegistration watchRegistration)
            throws InterruptedException {
        ReplyHeader r = new ReplyHeader();
        Packet packet = queuePacket(h, r, request, response, null, null, null,
                    null, watchRegistration);
        synchronized (packet) {
            while (!packet.finished) {
                packet.wait();
            }
        }
        return r;
    }
'''

For ZooKeeper asynchronous API, the first 3 steps are the same, after that, it'll call ClientCnxn::queuePacket(), and then return directly.

# References & todos
- [ ] ZooKeeper is using Haddop Record I/O for RequestHead/ReplyHead, Request, Response. http://www.tom-e-white.com/2008/07/rpc-and-serialization-with-hadoop.html
- [ ] ClientCnxn.java for ClientCnxn::queuePacket() internals.
- [ ] ZooKeeper::ZKWatchManager for watcher management. 

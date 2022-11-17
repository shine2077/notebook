# RTMP

## Ѱ��RTMP�������

![srs1](srs-1.png "srs-1")

�� `Srs`��`Srt`��`Rtc` ����ע�ᵽ`_srs_hybrid`����ʵ���ǰ�������������ӵ�һ�� `std::vector<ISrsHybridServer*> servers`

`_srs_hybrid->run()` �Ὺ��Э�̣�Ȼ��һֱ����

���� `_srs_hybrid->run()` ����

```c++
srs_error_t SrsHybridServer::run()
{
    srs_error_t err = srs_success;

    // Wait for all servers which need to do cleanup.
    SrsWaitGroup wg;

    vector<ISrsHybridServer*>::iterator it;
    for (it = servers.begin(); it != servers.end(); ++it) {
        ISrsHybridServer* server = *it;
        //����������������Srs��Srt��Rtc����
        if ((err = server->run(&wg)) != srs_success) {
            return srs_error_wrap(err, "run server");
        }
    }

    // Wait for all server to quit.
    wg.wait();

    return err;
}
```

������ֻ��ע`RTMP`������ص�`Srs`���������,��`SrsServerAdapter`�࣬�����UMLͼ����

![srs2](srs-2.png "srs-2")

`SrsServerAdapter` �� `run` ����ʵ�����£�

```c++
srs_error_t SrsServerAdapter::run(SrsWaitGroup* wg)
{
    srs_error_t err = srs_success;

    // Initialize the whole system, set hooks to handle server level events.
    if ((err = srs->initialize()) != srs_success) {
        return srs_error_wrap(err, "server initialize");
    }

    if ((err = srs->initialize_st()) != srs_success) {
        return srs_error_wrap(err, "initialize st");
    }

    if ((err = srs->initialize_signal()) != srs_success) {
        return srs_error_wrap(err, "initialize signal");
    }
    //��ʼ�����˿��ˣ�listen fd �ᱣ���ڶ������棬һ��Э�̼���һ��listen fd
    if ((err = srs->listen()) != srs_success) {
        return srs_error_wrap(err, "listen");
    }

    if ((err = srs->register_signal()) != srs_success) {
        return srs_error_wrap(err, "register signal");
    }

    if ((err = srs->http_handle()) != srs_success) {
        return srs_error_wrap(err, "http handle");
    }

    if ((err = srs->ingest()) != srs_success) {
        return srs_error_wrap(err, "ingest");
    }
    //����Srs����
    if ((err = srs->start(wg)) != srs_success) {
        return srs_error_wrap(err, "start");
    }
    //
    //ʡ���м�Ĵ���
    //
    return err;
}
```

`srs->listen()`����ʼ�����˿ڣ��������£�

```c++
srs_error_t SrsServer::listen()
{
    srs_error_t err = srs_success;

    // Create RTMP listeners.
    //��ȡ�����ļ���rtmp�����ip�Ͷ˿ںţ�����ӵ�������vector<SrsTcpListener*>
    rtmp_listener_->add(_srs_config->get_listens())->set_label("RTMP");
    //�����б��з�ip�Ͷ˿�
    if ((err = rtmp_listener_->listen()) != srs_success) {
        return srs_error_wrap(err, "rtmp listen");
    }
    //
    //ʡ���м�����Э��������ļ������Ĵ���ֻ��RTMPЭ���
    //

    if ((err = conn_manager->start()) != srs_success) {
        return srs_error_wrap(err, "connection manager");
    }

    return err;
}
```

����`rtmp_listener_->listen()`�Ĵ��룺

```c++
srs_error_t SrsMultipleTcpListeners::listen()
{
    srs_error_t err = srs_success;
    //���������б����ο�ʼ����
    for (vector<SrsTcpListener*>::iterator it = listeners_.begin(); it != listeners_.end(); ++it) {
        SrsTcpListener* l = *it;

        if ((err = l->listen()) != srs_success) {
            return srs_error_wrap(err, "listen");
        }
    }

    return err;
}
```

`l->listen()`���ú���`SrsTcpListener::listen()`����������

```c++
srs_error_t SrsTcpListener::listen()
{
    srs_error_t err = srs_success;

    // Ignore if not configured.
    if (ip.empty() || !port_) return err;

    srs_close_stfd(lfd);
    if ((err = srs_tcp_listen(ip, port_, &lfd)) != srs_success) {
        return srs_error_wrap(err, "listen at %s:%d", ip.c_str(), port_);
    }
    
    srs_freep(trd);
    //����Э��
    trd = new SrsSTCoroutine("tcp", this);
    if ((err = trd->start()) != srs_success) {
        return srs_error_wrap(err, "start coroutine");
    }

    int fd = srs_netfd_fileno(lfd);
    srs_trace("%s listen at tcp://%s:%d, fd=%d", label_.c_str(), ip.c_str(), port_, fd);
    
    return err;
}
```

��������ĵ�������ͼ

![srs3](srs-3.png "srs-3")

����ִ����֮���õ��˼������ļ�������lfd��ʼ����Э��

`trd->start()`���� SrsFastCoroutine::start() ����һ��Э�̡�

```c++
srs_error_t SrsFastCoroutine::start()
{
    srs_error_t err = srs_success;
    
    //����

    //����Э��
    if ((trd = (srs_thread_t)_pfn_st_thread_create(pfn, this, 1, stack_size)) == NULL) {
        err = srs_error_new(ERROR_ST_CREATE_CYCLE_THREAD, "create failed");
        
        srs_freep(trd_err);
        trd_err = srs_error_copy(err);
        
        return err;
    }
    
    started = true;

    return err;
}
```

`_pfn_st_thread_create()` ���ݵ��� pfn�� ����Э��ʵ�ʵ��õĺ����� `SrsFastCoroutine::pfn(void* arg)`

```c++
void* SrsFastCoroutine::pfn(void* arg)
{
    SrsFastCoroutine* p = (SrsFastCoroutine*)arg;

    srs_error_t err = p->cycle();

    // Set the err for function pull to fetch it.
    // @see https://github.com/ossrs/srs/pull/1304#issuecomment-480484151
    if (err != srs_success) {
        srs_freep(p->trd_err);
        // It's ok to directly use it, because it's returned by st_thread_join.
        p->trd_err = err;
    }

    return (void*)err;
}
```

`pfn()` ʵ���������ǵ���`SrsFastCoroutine`����� `cycle()` ��ѭ������ҵ��

```c++
srs_error_t SrsFastCoroutine::cycle()
{
    if (_srs_context) {
        if (cid_.empty()) {
            cid_ = _srs_context->generate_id();
        }
        _srs_context->set_id(cid_);
    }
    
    srs_error_t err = handler->cycle();
    if (err != srs_success) {
        return srs_error_wrap(err, "coroutine cycle");
    }

    // Set cycle done, no need to interrupt it.
    cycle_done = true;
    
    return err;
}
```

ʵ�����ǵ���`handler->cycle()`������`handler`Ϊ`ISrsCoroutineHandler`�࣬`SrsTcpListener`���������࣬ʵ�ʾ��ǵ���`SrsTcpListener::cycle()`

```c++
srs_error_t SrsTcpListener::cycle()
{
    srs_error_t err = srs_success;
    
    while (true) {
        if ((err = trd->pull()) != srs_success) {
            return srs_error_wrap(err, "tcp listener");
        }
        
        srs_netfd_t fd = srs_accept(lfd, NULL, NULL, SRS_UTIME_NO_TIMEOUT);
        if(fd == NULL){
            return srs_error_new(ERROR_SOCKET_ACCEPT, "accept at fd=%d", srs_netfd_fileno(lfd));
        }
        
        //ʡ���м����
        
        if ((err = handler->on_tcp_client(this, fd)) != srs_success) {
            return srs_error_wrap(err, "handle fd=%d", srs_netfd_fileno(fd));
        }
    }
    
    return err;
}
```
 rtmp ���ӵĴ�����ڣ����� new SrsRtmpConn()�������������ͼ

![srs4](srs-4.png "srs-4")

```c++
srs_error_t SrsServer::do_on_tcp_client(ISrsListener* listener, srs_netfd_t& stfd)
{
    srs_error_t err = srs_success;

    //ʡ��

    // Create resource by normal listeners.
    if (!resource) {
        if (listener == rtmp_listener_) {
            resource = new SrsRtmpConn(this, stfd2, ip, port);
        } 

    //ʡ��

    }

    // Use connection manager to manage all the resources.
    conn_manager->add(resource);

    // If connection is a resource to start, start a coroutine to handle it.
    ISrsStartable* conn = dynamic_cast<ISrsStartable*>(resource);
    if ((err = conn->start()) != srs_success) {
        return srs_error_wrap(err, "start conn coroutine");
    }

    return err;
}
```

## ����RTMP����

```c++
SrsRtmpConn::SrsRtmpConn(SrsServer* svr, srs_netfd_t c, string cip, int cport)
{
    // Create a identify for this client.
    _srs_context->set_id(_srs_context->generate_id());

    server = svr;

    stfd = c;
    skt = new SrsTcpConnection(c);
    manager = svr;
    ip = cip;
    port = cport;
    create_time = srsu2ms(srs_get_system_time());
    span_main_ = _srs_apm->dummy();
    span_connect_ = _srs_apm->dummy();
    span_client_ = _srs_apm->dummy();
    trd = new SrsSTCoroutine("rtmp", this, _srs_context->get_id());

    kbps = new SrsNetworkKbps();
    kbps->set_io(skt, skt);
    delta_ = new SrsNetworkDelta();
    delta_->set_io(skt, skt);
    
    rtmp = new SrsRtmpServer(skt);
    refer = new SrsRefer();
    security = new SrsSecurity();
    duration = 0;
    wakable = NULL;
    
    mw_sleep = SRS_PERF_MW_SLEEP;
    mw_msgs = 0;
    realtime = SRS_PERF_MIN_LATENCY_ENABLED;
    send_min_interval = 0;
    tcp_nodelay = false;
    info = new SrsClientInfo();

    publish_1stpkt_timeout = 0;
    publish_normal_timeout = 0;
    
    _srs_config->subscribe(this);
}
```

�ڶ�������`srs_netfd_t c` ���Ƕ�ԭʼ `tcp fd`�ķ�װ��`skt = new SrsTcpConnection(c)`����һ��Tcp���ӹ�����`skt`�������������`tcp fd`���ж�д

`new SrsSTCoroutine("rtmp", this, _srs_context->get_id())`������һ��Э������������ͻ��˵�RTMP���ӣ���Э��ͨ�����ú���`SrsRtmpConn::cycle()`��ѭ������`rtmp = new SrsRtmpServer(skt)`��TCP�����ϴ�����RTMP����

## ����RTMP����

![srs6](srs-6.png "srs-6")



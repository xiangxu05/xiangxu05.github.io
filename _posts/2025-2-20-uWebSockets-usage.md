---
layout: post
title: 学习笔记｜uWebSockets示例使用 如何使用C++17标准来编写ws示例
categories: [Study Notes]
description: uWebSockets开源库使用过程中遇到的一个问题，做一个记录。
keywords: uWebSockets, C++
mermaid: false
sequence: false
flow: false
mathjax: true
mindmap: false
mindmap2: false
---

​	uWebSockets是一个用C++17标准实现的网络库，底层使用的自行实现的uSockets做网络处理，是一个高性能、多平台兼容的开源项目。地址：[uWebSockets: Simple, secure & standards compliant web server for the most demanding of applications](https://github.com/uNetworking/uWebSockets)

## 示例代码

​	整个项目以C++17以下标准语法进行编写的，然而为了美观，示例中的初始化部分使用了C++20标准。

```c++
/* One app per thread; spawn as many as you have CPU-cores and let uWS share the listening port */
uWS::SSLApp({

    /* These are the most common options, fullchain and key. See uSockets for more options. */
    .cert_file_name = "cert.pem",
    .key_file_name = "key.pem"
    
}).get("/hello/:name", [](auto *res, auto *req) {

    /* You can efficiently stream huge files too */
    res->writeStatus("200 OK")
       ->writeHeader("Content-Type", "text/html; charset=utf-8")
       ->write("<h1>Hello ")
       ->write(req->getParameter("name"))
       ->end("!</h1>");
    
}).ws<UserData>("/*", {

    /* Just a few of the available handlers */
    .open = [](auto *ws) {
        ws->subscribe("oh_interesting_subject");
    },
    .message = [](auto *ws, std::string_view message, uWS::OpCode opCode) {
        ws->send(message, opCode);
    }
    
}).listen(9001, [](auto *listenSocket) {

    if (listenSocket) {
        std::cout << "Listening on port " << 9001 << std::endl;
    } else {
        std::cout << "Failed to load certs or to bind to port" << std::endl;
    }
    
}).run();
```

## 问题：初始化部分

​	可以看如下部分：

```c++
ws<UserData>("/*", {

    /* Just a few of the available handlers */
    .open = [](auto *ws) {
        ws->subscribe("oh_interesting_subject");
    },
    .message = [](auto *ws, std::string_view message, uWS::OpCode opCode) {
        ws->send(message, opCode);
    }
```

​	此处用到了带"."的初始化方法，仅对结构体中的部分内容赋值，而这种语法在C++20以前是不支持的。不巧的是有一些库，比如openCV并不支持C++20语法，在集成的时候会出现问题。因此需要对这个初始化方法进行修改。	

​	无论是SSLApp还是App对象，均是从TemplatedApp而来，在TemplatedApp中定义了ws对象的初始化方法，即传入一个结构体WebSocketBehavior。

```c++
    template <typename UserData>
    struct WebSocketBehavior {
        /* Disabled compression by default - probably a bad default */
        CompressOptions compression = DISABLED;
        /* Maximum message size we can receive */
        unsigned int maxPayloadLength = 16 * 1024;
        /* 2 minutes timeout is good */
        unsigned short idleTimeout = 120;
        /* 64kb backpressure is probably good */
        unsigned int maxBackpressure = 64 * 1024;
        bool closeOnBackpressureLimit = false;
        /* This one depends on kernel timeouts and is a bad default */
        bool resetIdleTimeoutOnSend = false;
        /* A good default, esp. for newcomers */
        bool sendPingsAutomatically = true;
        /* Maximum socket lifetime in minutes before forced closure (defaults to disabled) */
        unsigned short maxLifetime = 0;
        MoveOnlyFunction<void(HttpResponse<SSL> *, HttpRequest *, struct us_socket_context_t *)> upgrade = nullptr;
        MoveOnlyFunction<void(WebSocket<SSL, true, UserData> *)> open = nullptr;
        MoveOnlyFunction<void(WebSocket<SSL, true, UserData> *, std::string_view, OpCode)> message = nullptr;
        MoveOnlyFunction<void(WebSocket<SSL, true, UserData> *, std::string_view, OpCode)> dropped = nullptr;
        MoveOnlyFunction<void(WebSocket<SSL, true, UserData> *)> drain = nullptr;
        MoveOnlyFunction<void(WebSocket<SSL, true, UserData> *, std::string_view)> ping = nullptr;
        MoveOnlyFunction<void(WebSocket<SSL, true, UserData> *, std::string_view)> pong = nullptr;
        MoveOnlyFunction<void(WebSocket<SSL, true, UserData> *, std::string_view, int, int)> subscription = nullptr;
        MoveOnlyFunction<void(WebSocket<SSL, true, UserData> *, int, std::string_view)> close = nullptr;
    };

    //·········//

    template <typename UserData>
    TemplatedApp &&ws(std::string pattern, WebSocketBehavior<UserData> &&behavior) {
        //·········//
    }
```

​	在外部来构造这个WebSocketBehavior变量有一个小坑，构造方式如下：

```c++
    uWS::App::WebSocketBehavior<std::string> behavior;
    behavior.open = [this](auto *ws){ onOpen(ws); };
    behavior.message = [this](auto *ws, std::string_view message, uWS::OpCode opCode){ onMessage(ws, message, opCode);};
    behavior.close = [this](auto *ws, int code, std::string_view message){ onClose(ws, code, message);};
    
    this->ws<std::string>("/*", std::move(behavior))
        .listen(port, [&](auto *token)
                {
        if (token) {
            LOG_INFO("{0}: WebSocket server is listening on port {1}", "wsServer", port);
        } else {
            LOG_ERROR("{0}: Failed to listen on port {1}", "wsServer", port);
        } })
        .run();

```

​	重点就在这个结构体的定义：uWS::App::WebSocketBehavior<std::string> behavior;

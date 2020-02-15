---
title: Baby TShark
date: 2020-02-15 10:00:00 +0100
categories: debugging
---

I was made aware of Wireshark when I wanted to investigate certain HTTP requests to Elasticsearch. Wireshark is a network protocol analyzer with a GUI, while TShark is the equivalent CLI tool.

TShark has a lot of options and capabilities to get low-level network insights but what I wanted to do was pretty specific and simple: I wanted to intercept and monitor HTTP requests to a certain port on localhost.

`tshark` lets you do that, without the need to set up a proxy, which would be how I normally do this sort of thing.

As an example let's start a server locally:

    python -m SimpleHTTPServer 8000

Let's send a request:

    $ curl localhost:8000/hey
    <head>
    <title>Error response</title>
    </head>
    <body>
    <h1>Error response</h1>
    <p>Error code 404.
    <p>Message: File not found.
    <p>Error code explanation: 404 = Nothing matches the given URI.
    </body>

It fails, as expected. With `tshark` you can monitor HTTP requests to port 8000 like this:

    $ tshark -i lo0 -Y http.request 'tcp port 8000'
    Capturing on 'Loopback: lo0'
    5   0.000132    127.0.0.1 → 127.0.0.1    HTTP 137 GET /hey HTTP/1.1

Here's how you can customize which HTTP data you see:

    $ # Going to run curl -X POST localhost:8000/hey -d '{ "yo": true }' -H 'Content-Type: application/json' from another terminal
    $ tshark -i lo0 -Y http.request -T fields -e http.request.method -e http.request.uri -e http.file_data 'tcp port 8000'
    Capturing on 'Loopback: lo0'
    POST	/hey	{ "yo": true }

If you want to see the request headers and body, you can get most of that data with the `-O http,json` option:

    $ # Going to run curl -X POST localhost:8000/hey -d '{ "yo": true }' -H 'Content-Type: application/json' from another terminal
    $ tshark -i lo0 -Y http.request -O http,json 'tcp port 8000'
    Capturing on 'Loopback: lo0'
    Frame 5: 204 bytes on wire (1632 bits), 204 bytes captured (1632 bits) on interface 0
    Null/Loopback
    Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
    Transmission Control Protocol, Src Port: 54939, Dst Port: 8000, Seq: 1, Ack: 1, Len: 148
    Hypertext Transfer Protocol
        POST /hey HTTP/1.1\r\n
            [Expert Info (Chat/Sequence): POST /hey HTTP/1.1\r\n]
                [POST /hey HTTP/1.1\r\n]
                [Severity level: Chat]
                [Group: Sequence]
            Request Method: POST
            Request URI: /hey
            Request Version: HTTP/1.1
        Host: localhost:8000\r\n
        User-Agent: curl/7.54.0\r\n
        Accept: */*\r\n
        Content-Type: application/json\r\n
        Content-Length: 14\r\n
            [Content length: 14]
        \r\n
        [Full request URI: http://localhost:8000/hey]
        [HTTP request 1/1]
        File Data: 14 bytes
    JavaScript Object Notation: application/json
        Object
            Member Key: yo
                True value
                Key: yo

Read more about TShark in the [docs](https://www.wireshark.org/docs/man-pages/tshark.html) or just run `tshark --help`.

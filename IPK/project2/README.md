Simple HTTP Server
==================

This project is an implementation of HTTP Server using sockets.
To start a server, please use folowing syntax:

```
./ipkhttpserver [-p PORT] [-c CHUNK_SIZE] [-t DELAY]
```

As you can see you can specify PORT number where the server listens, set CHUNK size and define DELAY between CHUNKs.
By default these values are used:

| parameter  | default value |
| ---------- | ------------- |
| PORT       | 8080          |
| CHUNK_SIZE | 4096          |
| DELAY      | 0             |

Server follows HTTP/1.1 standard. That means it can be reached from any webbrowser currently on the market.

Content type
------------
In the task specification there was assumed sending of 'text/plain' content. This implemetation is capable of guessing mimetype and sends also different types of data.
Server has access to data in local subfolder `www` only.

Error handling
--------------

Server recognises two different error states. If the file is not found `404 Not Found` is returned . If the request is not valid HTTP request `400 Bad Request` is raised.
Response messages looks like this:

```
HTTP/1.1 404 Not Found\r\n
\r\n
```
or
```
HTTP/1.1 400 Bad Request\r\n
\r\n
```
Also when server is facing any kind of `ConnectionError` exception (including for example `BrokenPipeError`) when sending data it just logs the issue and continues in work (by sending next chunk or listening to the next request).

Chunk-based sending
-------------------

If the file itself is larger than CHUNK_SIZE it's split into chunks. The response of the server follows HTTP standard:

```
HTTP/1.1 200 OK\r\n
Content-Type: text/plain\r\n
Transfer-Encoding: chunked\r\n
\r\n
1000\r\n
Chunk content...\r\n
1000\r\n
Next chunk...\r\n

... etc. ...

0\r\n
\r\n
```
First of all header and first chunk are sent (together). After a delay server sends next chunk and repeats this step until end of file. The last chunk's size ussually differs from CHUNK_SIZE.

Simple sending
-------------
When it's not needed to send data in chunks. The content size is stated in header:
```
HTTP/1.1 200 OK\r\n
Content-Type: text/plain\r\n
Content-Length: 74\r\n
\r\n
```

Simple HTTP Client
==================

Get request sender. Launched with URI containing desired path/file.
```
./ipkhttpclient http://HOST:PORT/PATH_TO_FILE
```
All parts of URI must be specified.

The request follows HTTP standard and looks like this:
```
GET /some_file.txt HTTP/1.1\r\n
Accept: */*\r\n
Accept-Encoding: identity\r\n
Host: server_hostname\r\n
Connection: close\r\n
\r\n
```
After sending an request client waits for the response and saves it in the `ipkResp-YYYY-MM-DD:hh:mm:ss.payload` file and header to `ipkResp-YYYY-MM-DD:hh:mm:ss.header`

Implementation details
======================
Server (and client also) is writen in Python3 using modules `socket`, `socketserver`, `urllib.parse`, `argparse`, `os`, `logging`, `datetime`, `mimetypes` and  `time`.

Logging
=======
Both Server and Client have two log files, one for incoming and one for outgoing communication.

You can find them in the folder where these programs are launched and under names `ipkHttp(Server|Client)-YYYY-MM-DD:hh:mm:ss.(in|out).log`

Test Suite
==========
By `make tests` you can run some basic test that covers Error states, chunk-based sending, chunk-based recieving, non-text files. Also tests running server with dirretent setup (params).

Build
=====
Makefile is attached. Simple run `make` but it's probably not needed at all.
Also `make clean` cleans the mess caused by the logs.
# Apache Portable Runtime (APR) Threaded socket server
I have been playing around with APR for the last few days for a project I am working on. I couldn’t find an example of a threaded socket server using APR sockets and threads so here is one that will hopefully help you.

Big thanks to [http://dev.ariel-networks.com/apr/apr-tutorial/html/apr-tutorial.html](http://dev.ariel-networks.com/apr/apr-tutorial/html/apr-tutorial.html) most of the code comes from his tutorials

```
#include <stdio.h>
#include <stdlib.h>
 
//APR Includes
#include <apr_general.h>
#include <apr_pools.h>
#include <apr_file_io.h>
#include <apr_strings.h>
#include <apr_network_io.h>
#include <apr_thread_proc.h>
 
/* default listen port number */
#define DEF_LISTEN_PORT     8081
 
/* default socket backlog number. SOMAXCONN is a system default value */
#define DEF_SOCKET_BACKLOG  SOMAXCONN
 
/* default buffer size */
#define BUFSIZE         4096
 
static void* APR_THREAD_FUNC processConnection(apr_thread_t *thd, void *);
 
int listenServer(){
    //Setup a socket to listen on the address for incoming requests
 
    apr_socket_t *listenSocket;
    apr_pool_t *memPool;
    apr_status_t retStatus;
    apr_threadattr_t *thd_attr;
    apr_sockaddr_t *sa;
 
    apr_pool_create(&memPool, NULL);
    apr_threadattr_create(&thd_attr, memPool);
 
    retStatus = apr_sockaddr_info_get(&sa, NULL, APR_INET, DEF_LISTEN_PORT, 0, memPool);
    if (retStatus != APR_SUCCESS) {
        goto error;
    }
 
    retStatus = apr_socket_create(&listenSocket, sa->family, SOCK_STREAM, APR_PROTO_TCP, memPool);
    if (retStatus != APR_SUCCESS) {
        goto error;
    }
 
    apr_socket_opt_set(listenSocket, APR_SO_NONBLOCK, 0);
    apr_socket_timeout_set(listenSocket, -1);
    apr_socket_opt_set(listenSocket, APR_SO_REUSEADDR, 1);
 
    retStatus = apr_socket_bind(listenSocket, sa);
    if (retStatus != APR_SUCCESS) {
        goto error;
    }
    retStatus = apr_socket_listen(listenSocket, DEF_SOCKET_BACKLOG);
    if (retStatus != APR_SUCCESS) {
        goto error;
    }
 
    while (1) {
 
        apr_socket_t *ns;/* accepted socket */
 
        retStatus = apr_socket_accept(&ns, listenSocket, memPool);
        if (retStatus != APR_SUCCESS) {
            goto error;
        }
 
        apr_socket_opt_set(ns, APR_SO_NONBLOCK, 0);
        apr_socket_timeout_set(ns, -1);
 
        //Create the new thread
        apr_thread_t *thd_obj;
        retStatus = apr_thread_create(&thd_obj, NULL, processConnection, ns, memPool);
 
        if(retStatus != APR_SUCCESS){
            printf("Error Creating new Thread\n");
        }
 
    }
 
    apr_pool_destroy(memPool);
    apr_terminate();
    return 0;
 
    error:
    {
        char errbuf[256];
        apr_strerror(retStatus, errbuf, sizeof(errbuf));
        printf("error: %d, %s\n", retStatus, errbuf);
    }
 
    apr_terminate();
    return -1;
}
 
static void* APR_THREAD_FUNC processConnection(apr_thread_t *thd, void* data){
 
    apr_socket_t * sock = (apr_socket_t*) data;
 
    while (1) {
        char buf[BUFSIZE];
        apr_size_t len = sizeof(buf) - 1;/* -1 for a null-terminated */
 
        apr_status_t rv = apr_socket_recv(sock, buf, &len);
 
        if (rv == APR_EOF || len == 0) {
            printf("Socket Closed\n");
            apr_socket_close(sock);
            break;
        }
 
        if(len > 0){
            printf("Read: %s\n", buf);
        }
 
        buf[len] = '\0';/* apr_socket_recv() doesn't return a null-terminated string */
 
    }
 
}
```
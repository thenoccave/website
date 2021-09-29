# Run kubernetes registry-creds through a proxy
Recently we had the need to run [registry-creds](https://github.com/upmc-enterprises/registry-creds) through a proxy to access ECR. The good news is that it respects the HTTP_PROXY, HTTPS_PROXY and NO_PROXY environment variables so it is just a matter of adding them to the deployment. 

You will need to NO_PROXY the internal IP used to access the API server otherwise it won't be able to connect to it.

```
containers:
      - env:
        - name: HTTP_PROXY
          value: http://<host>:<port>
        - name: HTTPS_PROXY
          value: http://<host>:<port>
        - name: NO_PROXY
          value: 10.96.0.1
```
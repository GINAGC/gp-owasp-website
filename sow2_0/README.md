
## Proxy configuration for owasp.org

## Requirements

- **Ubuntu LTS**

- **Docker**


## Installation

- Setup Docker

- Prepare the repositories
  
  ```bash
    git clone --recursive https://github.com/k8-proxy/k8-reverse-proxy
    wget https://github.com/filetrust/Glasswall-Rebuild-SDK-Evaluation/raw/master/Linux/Library/libglasswall.classic.so -O k8-reverse-proxy/stable-src/c-icap/Glasswall-Rebuild-SDK-Evaluation/Linux/Library/libglasswall.classic.so
  ```
- Copy required files specific to owasp.org site

 ```bash
  cp openssl.cnf k8-reverse-proxy/stable-src/
  cp *.sh k8-reverse-proxy/stable-src/
  cp gwproxy.env k8-reverse-proxy/stable-src/

 ```

- Move to k8-reverse-proxy : `cd k8-reverse-proxy/stable-src/`

- Generate new SSL credentials
  
  ```bash
    ./gencert.sh
    mv full.pem nginx/
  ```

- Start the deployment    
  
  ```bash
    docker-compose up -d --build
  ```

  ## Client configuration

- Add hosts records to your client system hosts file:
  ```bash
  127.0.0.1 owasp.org.glasswall-icap.com
  ```

  ## Access the proxied site
  
  You can access the proxied site by browsing [owasp.org.glasswall-icap.com](https://owasp.org.glasswall-icap.com) after adding `k8-reverse-proxy/stable-src/server.crt` to your browser/system ssl trust store.


## Proxy configuration for owasp.org

## Requirements

- **Docker**

- **Kubernetes**

- **Helm 3**

## Installation with deployed icap server
1. kubectl create namespace icap-adaptation
2. git clone https://github.com/anejaalekh/gp-owasp-website.git
3. git clone https://github.com/k8-proxy/k8-reverse-proxy.git --recursive && cd k8-reverse-proxy/stable-src/
4. Update gwproxy.env with website details i.e ALLOWED_DOMAINS, SUBFILTER_ENV
5. Replcae nginx/full.pem ../../gp-owasp-website/sow2_0/full.pem (gp-owasp-website is cloned in step 2)
6. Build images for squid and nginx. Optionally Images can be pushed to docker repository as well.
    ```
    docker build nginx -t <docker registry>/reverse-proxy-nginx:0.0.1
    docker build squid -t <docker registry>/reverse-proxy-squid:0.0.1

7. git clone https://github.com/k8-proxy/s-k8-proxy-rebuild.git && cd s-k8-proxy-rebuild/stable-src/  

8. Update chart/values  
	```
	set service.icap.enabled=false
	set service.nginx.additionalHosts 
	    - www.owasp.org.glasswall-icap.com
	    - cse.google.com.glasswall-icap.com
	    - www.cse.google.com.glasswall-icap.com
	    - google.com.glasswall-icap.com
	    - www.google.com.glasswall-icap.com
	set service.nginx.url=owasp.org.glasswall-icap.com
	
9. Deploy the helm chart
	```
	helm --namespace icap-adaptation upgrade --install \
	  --set image.nginx.repository=glasswallsolutions/reverse-proxy-nginx \
	  --set image.nginx.tag=0.0.1 \
	  --set image.squid.repository=glasswallsolutions/reverse-proxy-squid \
	  --set image.squid.tag=0.0.1 \
	  --set application.nginx.env.ICAP_URL=icap://54.77.168.168:1344/gw_rebuild \
	  --set application.squid.env.ICAP_URL=icap://54.77.168.168:1344/gw_rebuild \
	  --set application.nginx.env.ALLOWED_DOMAINS='owasp.org.glasswall-icap.com\,www.owasp.org.glasswall-icap.com\,cse.google.com.glasswall-icap.com\,www.cse.google.com.glasswall-icap.com\,google.com.glasswall-icap.com\,www.google.com.glasswall-icap.com' \
	  --set application.nginx.env.ROOT_DOMAIN='glasswall-icap.com' \
	  --set application.nginx.env.SUBFILTER_ENV='owasp.org\,owasp.org.glasswall-icap.com cse.google.com\,cse.google.com.glasswall-icap.com google.com\,google.com.glasswall-icap.com' \
	  --set application.squid.env.ALLOWED_DOMAINS='owasp.org.glasswall-icap.com\,www.owasp.org.glasswall-icap.com\,cse.google.com.glasswall-icap.com\,www.cse.google.com.glasswall-icap.com\,google.com.glasswall-icap.com\,www.google.com.glasswall-icap.com' \
	  --set application.squid.env.ROOT_DOMAIN='glasswall-icap.com' \
	  reverse-proxy chart/
	  
## Verify the installation

1. kubectl get ing -n icap-adaptation (copy the name of ingress name)
2. kubectl -n icap-adaptation edit ing <ingress_name_from_step1>
3. check spec->rules->host  and spec->tls->hosts check the entries here
4. kubectl get deployments -n icap-adaptation
5. verify nginx/squid deployment configuration i.e spec->containers->env . properties to check ICAP_URL, ALLOWED_DOMAINS, and SUBFILTER_ENV
    
  

## Installation with local deployed icap server

 1. kubectl create namespace icap-adaptation
 2. kubectl create -n icap-adaptation secret docker-registry regcred --docker-server='https://index.docker.io/v1/' --docker-username=<username> --docker-password=<passwd> --docker-email=email
 3. git clone https://github.com/anejaalekh/gp-owasp-website.git
 4. cd gp-owasp-website/sow2_0/adaptation/
 5. helm install . --namespace icap-adaptation --generate-name
 6. kubectl get pods -n icap-adaptation, Check Pods status should be Running. In case if icap-adaptation pod is not running in error state, delete the pod 
 
 **Prepare Reverse Proxy Images**
 
 7. git clone https://github.com/k8-proxy/k8-reverse-proxy.git --recursive && cd k8-reverse-proxy/stable-src/
 8. Update gwproxy.env with website details
 9. Replcae nginx/full.pem ../../gp-owasp-website/sow2_0/full.pem (gp-owasp-website is cloned in step 3)
 9. Build reverse proxy images
 
		 docker build nginx -t <docker registry>/reverse-proxy-nginx:0.0.1
		 docker push <docker registry>/reverse-proxy-nginx:0.0.1  # Optional

		 docker build squid -t <docker registry>/reverse-proxy-squid:0.0.1
		 docker push <docker registry>/reverse-proxy-squid:0.0.1 # Optional

		 wget -O c-icap/Glasswall-Rebuild-SDK-Evaluation/Linux/Library/libglasswall.classic.so https://github.com/filetrust/Glasswall-Rebuild-SDK-Evaluation/releases/download/1.117/libglasswall.classic.so
		 docker build c-icap -t <docker registry>/reverse-proxy-c-icap:0.0.1
		 docker push <docker registry>/reverse-proxy-c-icap:0.0.1  # Optional
 10. kubectl -n icap-adaptation get svc | grep icap-service, save this value 
 
 **Install Reverse Proxy**
 
 11. git clone https://github.com/k8-proxy/s-k8-proxy-rebuild.git && cd s-k8-proxy-rebuild/stable-src/
 12. Update website info in charts/values.yaml file (Icap url can also be updated directly in charts/templates/deployment.yaml with value from step 10, search for ICAP_URL) 
 13. Install nginx, squid reverse proxy 
 
	helm --namespace icap-adaptation upgrade --install \
	  --set image.nginx.repository=<docker registry>/reverse-proxy-nginx \
	  --set image.nginx.tag=0.0.1 \
	  --set image.squid.repository=<docker registry>/reverse-proxy-squid \
	  --set image.squid.tag=0.0.1 \
	  --set image.icap.repository=<docker registry>/reverse-proxy-c-icap \
	  --set image.icap.tag=0.0.1 \
	  --set application.nginx.env.ALLOWED_DOMAINS='owasp.org.glasswall-icap.com\,www.owasp.org.glasswall-icap.com,cse.google.com.glasswall-icap.com,www,cse.google.com.glasswall-icap.com,google.com.glasswall-icap.com,www,google.com.glasswall-icap.com' \
	  --set application.nginx.env.ROOT_DOMAIN='glasswall-icap.com' \
	  --set application.nginx.env.SUBFILTER_ENV='owasp.org\,owasp.org.glasswall-icap.com cse.google.com,cse.google.com.glasswall-icap.com google.com google.com.glasswall-icap.com' \
	  --set application.squid.env.ALLOWED_DOMAINS='owasp.org.glasswall-icap.com\,www.owasp.org.glasswall-icap.com,cse.google.com.glasswall-icap.com,www,cse.google.com.glasswall-icap.com,google.com.glasswall-icap.com,www,google.com.glasswall-icap.com' \
	  --set application.squid.env.ROOT_DOMAIN='glasswall-icap.com' \
	  reverse-proxy chart/
  
 14. Verify the website values for nginx ingress, squid and nginx deployment.
 15. Update the ICAP URL in squid and nginx deployment with value from step 10, if not updated already 
 16. Verify all the pods in icap-adaptation namespace
 17. sudo kubectl -n icap-adaptation port-forward --address 0.0.0.0 svc/reverse-proxy-reverse-proxy-nginx 443:443
 
 
 **Note:**
 1. When request is sent from browser, verify that rebuild pods are created and in Running status. Sometimes they are not able to pull images. Manually pull the images or docker login solves the problem. 
 2. Verify ingress details as well, host and path details should be proper.
 3. https://github.com/filetrust/icap-infrastructure has issues(unbounded volume claim), so adaptation folder content is modified and placed inside sow2_0/adaptation folder

## OWASP Search Scenario
In owasp website search results were not working properly as Owasp use JS functions from cse.google.com and google.com. Search results were fetched by JS. As this was done on browser and we were proxing owasp.org url. Due to this search results urls were not rewritten. As a fix we are rewitting cse.google.com and google.com. Updated allowed domains are:  owasp.org.glasswall-icap.com,www.owasp.org.glasswall-icap.com,cse.google.com.glasswall-icap.com,www.cse.google.com.glasswall-icap.com,google.com.glasswall-icap.com,www.google.com.glasswall-icap.com


## Proxy configuration for owasp.org

## Requirements

- **Docker**

- **Kubernetes**

- **Helm**


## Installation

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
	  --set application.nginx.env.ALLOWED_DOMAINS='owasp.org.glasswall-icap.com\,www.owasp.org.glasswall-icap.com' \
	  --set application.nginx.env.ROOT_DOMAIN='glasswall-icap.com' \
	  --set application.nginx.env.SUBFILTER_ENV='owasp.org\,owasp.org.glasswall-icap.com' \
	  --set application.squid.env.ALLOWED_DOMAINS='owasp.org.glasswall-icap.com\,www.owasp.org.glasswall-icap.com' \
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





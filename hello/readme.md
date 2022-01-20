# Simple docker helloworld

It will run:
- A nginx container
- On port 8080

To use:
- Build: `docker build -t nugie/hello-nugie`
- Run: `docker run --name hello -p 8080:8080 nugie/hello-nugie`
- Curl localhost:8080

Kubernetes:
- Simple hello: `kubectl apply -f hello.yml`
- Test the app by curling the public DNS `EXTERNAL-IP` listed here: `kubectl get svc`
- Curl: `curl [hostname_publik]:8080`
- Delete deploy: `kubectl delete -f hello.yml`
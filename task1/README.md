RUN:
docker-compose up -d build

CHECK: curl http://localhost:8181
curl http://localhost
curl --insecure https://localhost

# Steps to deploy votingapp

1.  **Create votingapp namespace**
- ```
  k create ns votingapp && \
  k config set-context --current --namespace=votingapp
  ```

2. **Create manifests for db-service**
- ```
  k create deployment -n votingapp db --image=postgres:16-alpine --port 5432 && \
  k set env -n votingapp deployment/db POSTGRES_USER=postgres POSTGRES_PASSWORD=postgres --dry-run=client -o yaml > db/deployment.yaml
  ```
- Create a `PersistentVolumeClaim` manifest referring the default `StorageClass`
- Edit the deployment manifest to add the `PersistentVolumeClaim` volume to /var/lib/postgresql/data mountPath
- ```
  k expose -f db/deployment.yaml --port=5432 --target-port=5432 --dry-run=client -o yaml > db/service.yaml && \
  k apply -f db/.
  ```

3. **Create manifests for redis-service**
- `k create deployment -n votingapp redis --image=redis:alpine --dry-run=client -o yaml > redis/deployment.yaml`
- Edit the generated manifest to add an emptyDir volume named "redis-data" to /data mountPath
- ```
  k expose -f redis/deployment.yaml --port=6379 --target-port=6379 --dry-run=client -o yaml > redis/service.yaml && \
  k apply -f redis/.
  ```

4. **Create manifests for worker-service**
- `k create deployment -n votingapp worker --image=dockersamples/examplevotingapp_worker --dry-run=client -o yaml | tee worker/deployment.yaml | k apply -f -`
- Change the default container image ENTRYPOINT and CMD

5. **Create manifests for vote-service**
    ```
    k create deployment -n votingapp vote --image=dockersamples/examplevotingapp_vote --port=80 --dry-run=client -o yaml > vote/deployment.yaml && \
    k expose -f vote/deployment.yaml --port=80 --target-port=80 --dry-run=client -o yaml > vote/service.yaml && \
    k create ingress vote --rule="vote.conquerproject.io/*=vote:80" --class=nginx --dry-run=client -o yaml > vote/ingress.yaml && \
    k apply -f vote/.
    ```
- Change the default container image ENTRYPOINT and CMD

6. **Create manifests for result-service**
    ```
    k create deployment -n votingapp result --image=dockersamples/examplevotingapp_result --port=80 --dry-run=client -o yaml > result/deployment.yaml && \
    k expose -f result/deployment.yaml --port=80 --target-port=80 --dry-run=client -o yaml > result/service.yaml && \
    k create ingress result --rule="result.conquerproject.io/*=result:80" --class=nginx --dry-run=client -o yaml > result/ingress.yaml && \
    k apply -f result/.
    ```
- Change the default container image ENTRYPOINT and CMD
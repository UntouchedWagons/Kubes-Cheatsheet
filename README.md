# Kubes Cheatsheet
A quick reference to useful Kubernetes commands

## Pods

View all pods

	kubectl get pods <-n namespace>

Get logs from pod

	kubectl logs pod-name <-c container> <-n namespace>

Get pod's resource usage

	kubectl top pod <pod-name> <-n namespace>

## Containers

Set a resources limit for a container

    ---
    kind: Deployment
    apiVersion: apps/v1
    metadata:
      name: ...
      namespace: testing
      labels:
        app: sonarr
    spec:
      replicas: 1
      progressDeadlineSeconds: 600
      revisionHistoryLimit: 0
      strategy:
        type: Recreate
      selector:
        matchLabels:
          app: sonarr
      template:
        metadata:
          labels:
            app: sonarr
        spec:
          containers:
            - name: ...
              image: ...
              resources:
                limits:
                  memory: 512Mi
                  cpu: "1"
                requests:
                  memory: 128Mi
                  cpu: "0.2"

CPU is measured in milli-units. 1000m equals 1 CPU core

## Secrets

    ---
    apiVersion: v1
    kind: Secret
    metadata:
      name: mariadb-server-secret
      namespace: testing
    type: Opaque
    stringData:
      MARIADB_ROOT_PASSWORD: AppleSauce
    ---
    apiVersion: v1
      kind: Pod
      metadata:
        name: ...
      spec:
        containers:
          - name: ...
            image: ...
            env:
              - name: MARIADB_ROOT_PASSWORD
                valueFrom:
                  secretKeyRef:
                    name: mariadb-server-secret
                    key: MARIADB_ROOT_PASSWORD

## Volumes

    ---
    kind: Deployment
    apiVersion: apps/v1
    metadata:
      name: ...
      namespace: testing
      labels:
        app: sonarr
    spec:
      replicas: 1
      progressDeadlineSeconds: 600
      revisionHistoryLimit: 0
      strategy:
        type: Recreate
      selector:
        matchLabels:
          app: sonarr
      template:
        metadata:
          labels:
            app: sonarr
        spec:
          volumes:
            - name: sonarr-config
              nfs:
                server: 192.168.5.5
                path: /mnt/nvme/K3S-Testing
          containers:
            - name: ...
              image: ...
              volumeMounts:
                - mountPath: /config
                  name: sonarr-config
                  subPath: sonarr

## Files

Copy file out of pod (specify `-c CONTAINER_NAME` if there's multiple containers in the pod)

    kubectl cp {NAMESPACE}/{POD_NAME}:/path/to/file file

## Running commands in a container

Runs the command `whoami` in the specified pod (specify `-c CONTAINER_NAME` if there's multiple containers in the pod)

    kubectl exec -it {POD_NAME} -- whoami

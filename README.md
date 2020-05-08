# helm-camunda-postgres

# Intstall Camunda with POSTGRESQL

This is a basic setup of Camunda using Helm. Currently Postgres is installed within a Pod but not used for the camunda application. The next steps is to make Camunda connect to the postgres db.

## Requirements: 
```
Install Minikube (v1.9.2 was used as part of this work)
Install Helm (v3.2.0 was used as part of this work)
```

## Commands:

```
Find minikube ip with command `minikube ip`

Edit Hosts file /etc/hosts using ip found
192.x.x.x camunda.minikube.local
```

```
minikube start
helm install <Release Name> <Camunda Chart>  (e.g. helm install dev ./camunda-install, you can replace dev with --generate-name)

kubectl get all (List Pod details)

To install postgress with camunda (by default it is disabled)
helm install dev ./camunda-install --set tags.database=true (or replace database with postgres either works)
```

Visiting camunda.minikube.local/camunda will take you to the camunda webpage or camunda.minikube.local to the Tomcat page
Alternativly use the ip address 192.x.x.x:30000 (Node port is set in the values file and fixed, if you wish to have a random nodeport remove the node port from config, at present this may break the code due to the dependency (NOT TESTED AS GOT LATE IN DAY TO WORK))


## Connect to DB

```
kubectl exec -ti postgres-1234 -- psql -h <POD IP> -p 30001 -U admin --password

postgres-1234 = POD NAME
30001 = NODE PORT
```
## TO DO:
```
Does not of probe for POSTGRES, so dumping notes below.
Notes dump:
          livenessProbe:
            exec:
              command:
                - /bin/sh
                - -c
                {{- if (include "postgres.database" .) }}
                - exec pg_isready -U {{ include "postgres.username" . | quote }} -d {{ (include "postgres.database" .) | quote }} -h 127.0.0.1 -p {{ template "postgres.port" . }}
                {{- else }}
                - exec pg_isready -U {{ include "postgres.username" . | quote }} -h 127.0.0.1 -p {{ template "postgres.port" . }}
                {{- end }}
            initialDelaySeconds: 30
          readinessProbe:
            httpGet:
              path: /
              port: http
            initialDelaySeconds: 30
```
```
Camunda Secrets

        env:
          - name: DB_URL
          - name: DB_USERNAME
            valueFrom:
              secretKeyRef:
                name: dev-postgres-secret
                key: postgres-username
          - name: DB_PASSWORD
            valueFrom:
              secretKeyRef:
                name: dev-postgres-secret
                key: postgres-password
          - name: DB_DRIVER
            value: org.postgresql.Driver
```

 volumes:
- name: camunda-pgdata
 persistentVolumeClaim:
            claimName: {{ include "postgres.fullname" . }}-pvc
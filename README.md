# handsonstatefullset

accès **interne** ou **externe**

---

## 1. Accès depuis l’intérieur du cluster (entre Pods)

Ton **Service Headless** (`mysql`) te donne déjà une **DNS interne**.

* Par exemple :

  * `mysql-0.mysql` → le pod maître
  * `mysql-1.mysql` et `mysql-2.mysql` → les réplicas

Depuis un autre Pod dans le cluster, tu peux faire :

```bash
mysql -h mysql-0.mysql -u root -p
```

C’est la méthode utilisée par tes **applications dans Kubernetes** pour se connecter.

---

## 2. Accès en local (administration / test)

Tu as plusieurs options :

### a) `kubectl exec`

Connexion directe dans un Pod MySQL :

```bash
kubectl exec -it mysql-0 -- mysql -u root -p
```

### b) `kubectl port-forward`

Tu exposes **ton Pod** localement :

```bash
kubectl port-forward pod/mysql-0 3306:3306
```

Puis tu te connectes depuis ton PC :

```bash
mysql -h 127.0.0.1 -P 3306 -u root -p
```

Ça marche bien pour l’administration, mais pas pour une application de prod.

---

## 3. Accès depuis l’extérieur du cluster (application hors K8s)

Si ton appli n’est **pas dans Kubernetes**, tu dois exposer MySQL :

### a) Service de type **NodePort**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-external
spec:
  type: NodePort
  ports:
    - port: 3306
      targetPort: 3306
      nodePort: 30306   # Port ouvert sur chaque Node
  selector:
    app: mysql
```

Connexion :

```bash
mysql -h <IP_NODE> -P 30306 -u root -p
```

### b) Service de type **LoadBalancer** (cloud : GKE, EKS, AKS…)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-lb
spec:
  type: LoadBalancer
  ports:
    - port: 3306
      targetPort: 3306
  selector:
    app: mysql
```

Un **IP public** sera attribué automatiquement.

---

# Automatic SSL Certificates with cert-manager

This directory contains ClusterIssuer configurations for automatic SSL certificate management using Let's Encrypt.

## ClusterIssuers Available

- **letsencrypt-staging**: For testing (certificates not trusted by browsers)
- **letsencrypt-production**: For production (real trusted certificates, has rate limits)

## How to Use

### 1. Update Email Address
Edit both `letsencrypt-staging.yaml` and `letsencrypt-production.yaml` and change:
```yaml
email: admin@example.com  # <- Change this to your email
```

### 2. Create an Ingress with Auto SSL
Add these annotations to any Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
  annotations:
    kubernetes.io/ingress.class: traefik
    cert-manager.io/cluster-issuer: letsencrypt-staging  # or letsencrypt-production
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls  # cert-manager will create this secret automatically
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

### 3. cert-manager Will Automatically:
1. Request a certificate from Let's Encrypt
2. Complete the HTTP-01 challenge
3. Store the certificate in the specified secret
4. Renew the certificate before expiration

## Switching from Staging to Production

Once you've tested with staging and everything works:
1. Change the annotation: `cert-manager.io/cluster-issuer: letsencrypt-production`
2. Delete the old secret: `kubectl delete secret myapp-tls -n namespace`
3. cert-manager will create a new production certificate

## Monitoring

Check certificate status:
```bash
kubectl get certificate -A
kubectl describe certificate <name> -n <namespace>
```

Check cert-manager logs:
```bash
kubectl logs -n cert-manager -l app=cert-manager
```

- [Kubernetes Data Encryption Config and Key](#kubernetes-data-encryption-config-and-key)
  - [What is the Data Encryption Config?](#what-is-the-data-encryption-config)
  - [Generating the Data Encryption Config](#generating-the-data-encryption-config)

## Kubernetes Data Encryption Config and Key
Notes taken from Linux Academy's Kubernetes the Hard Way course.

### What is the Data Encryption Config?
* K8s supports the ability to encrypt secret data at rest
* This means secrets are encrypted and are never stored in plain text
* We need to generate a key and place it in a configuration file to be copied to the controller nodes
* More info: [https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/]

### Generating the Data Encryption Config
* First, we can use the following code to generate an encryption key and store it in a config file:
```bash
# Create and store the key in a base64-encoded environment variable
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)

# Create the config file
cat > encryption-config.yaml << EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```
* We can then upload the config file to our controller nodes using `scp`:
```bash
scp encryption-config.yaml user@<controller 1 public ip>:~/
scp encryption-config.yaml user@<controller 2 public ip>:~/
```
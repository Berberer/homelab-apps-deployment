## Personal Homelab Apps Deployment

### Prerequisites
- `kubeconfig` for the cluster at `~/.kube/config`
- Private Age key at `~/.config/age/key.txt`
- Install the cert-manager components:
    ```bash
    kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.18.2/cert-manager.yaml
    ```

### Deployment
1. Create the namespace:
    ```bash
    kubectl create namespace homelab
    ```
2. Deploy the base components:
    ```bash
    kubectl apply -k ./kustomize/overlays/base
    ```
3. Execute the database setup:
    1. Create the required databases (Memos, Linkding):
        ```sql
        CREATE DATABASE <db>;
        ```
    2. For each new database, login and execute the user setup:
        ```sql
        CREATE ROLE <username> WITH LOGIN ENCRYPTED PASSWORD '<password>';

        ALTER DATABASE <db> OWNER TO <username>;

        ALTER SCHEMA public OWNER TO <username>;

        GRANT ALL ON SCHEMA public TO <username>;

        ALTER DEFAULT PRIVILEGES IN SCHEMA public
            GRANT ALL ON TABLES TO <username>;
        ALTER DEFAULT PRIVILEGES IN SCHEMA public
            GRANT ALL ON SEQUENCES TO <username>;
        ALTER DEFAULT PRIVILEGES IN SCHEMA public
            GRANT ALL ON FUNCTIONS TO <username>;
        ```
4. Deploy the app components:
    ```bash
    kubectl apply -k ./kustomize/overlays/apps
    ```

### Secrets
Encrypt a secrets file:
```bash
sops encrypt --age <public-key> --input-type binary --output <path>.enc <path>
```
The secrets should be automatically decrypted via Direnv on entering the directory. If Direnv is not used, use this command for decrypting secrets:
```bash
SOPS_AGE_KEY_FILE=~/.config/age/key.txt sops decrypt --output <path> <path>.enc
```

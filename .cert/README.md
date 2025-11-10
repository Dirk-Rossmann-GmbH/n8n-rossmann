# Rossmann SSL Certificates# SSL Certificates



This folder contains Rossmann's self-signed SSL certificates required for accessing internal corporate services from within Docker containers.This folder contains ROSSMANN's self-signed SSL certificates required for corporate network access.



## Included Files## Included Files



- ✅ `root.crt` - Root certificate authority- ✅ `root.crt` - Root certificate authority

- ✅ `rms-intern.crt` - ROSSMANN internal certificate- ✅ `rms-intern.crt` - ROSSMANN internal certificate



These certificates are **automatically installed** during Docker image build.These certificates are **already included** in the template and will be automatically installed during container build.



## How It Works## How It Works



The `Dockerfile.n8n-custom` copies these certificates and installs them:The Dockerfile copies these certificates to the system's trusted certificate store:



```dockerfile```dockerfile

COPY .cert/*.crt /usr/local/share/ca-certificates/COPY .cert/*.crt /usr/local/share/ca-certificates/

RUN cat /usr/local/share/ca-certificates/*.crt >> /etc/ssl/certs/ca-certificates.crt && \RUN update-ca-certificates

    update-ca-certificates```

```

This enables:

This enables n8n to:

- Access to internal ROSSMANN services

- Access internal Rossmann services- Azure OpenAI API calls through corporate network

- Connect to Azure OpenAI through corporate network- Internal package registries

- Use internal package registries- Git operations with internal repositories

- Make HTTPS calls to internal APIs without certificate errors

## Security Note

## Certificate Updates

⚠️ **Important for Internal Use Only**

When Rossmann renews the certificates:

- These certificates are for ROSSMANN internal projects only

1. Replace the `.crt` files in this directory- Do NOT share this template externally or commit to public repositories

2. Rebuild the Docker image:- Keep the template in internal/private repositories only


   ```bash
   # Windows
   wsl docker build -f Dockerfile.n8n-custom -t n8n-custom .

   # Linux/WSL
   docker build -f Dockerfile.n8n-custom -t n8n-custom .
   ```

3. Restart the containers:

   ```bash
   wsl docker-compose -f docker-compose.rossmann.yml restart
   ```

## Security Note

⚠️ **For Internal Use Only**

- These certificates are for Rossmann internal projects
- Do NOT share externally or commit to public repositories
- Keep in private/internal repositories only

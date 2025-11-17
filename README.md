# Apheresis Cell Therapy Analytics

A predictive analytics platform for autologous cell therapy manufacturing that analyzes apheresis material to forecast key performance indicators including time to dose, harvest day success probability, and critical quality attributes (CQAs) of the final drug product.

## Overview

This application combines Django backend services with Ollama LLM capabilities and Neo4j graph database to provide comprehensive analysis of cell therapy manufacturing data.

## Architecture

- **Backend**: Django application with SQLite database
- **LLM Service**: Ollama with all-minilm and gemma:2b models
- **Graph Database**: Neo4j 5.19.0 with APOC plugins
- **Networking**: Custom Docker network for inter-service communication

## Prerequisites

- Docker and Docker Compose
- NVIDIA GPU with Docker GPU support (for Ollama)
- SSL certificates (`cert.pem` and `privkey.pem`) in project root

## Quick Start

### 1. Pull Required Images

```bash
docker pull ollama/ollama
```

### 2. Build Application Images

```bash
# Production image
docker build -t grant_app .

# Development image
docker build -t grant_app_dev .
```

### 3. Create Docker Network

```bash
docker network create django-ollama-service
```

### 4. Launch Services

**Production:**
```bash
docker run --network=django-ollama-service \
  -p 443:443 -p 80:80 \
  --name django \
  grant_app
```

**Development:**
```bash
docker run --network=django-ollama-service \
  -p 8080:80 -p 4343:443 \
  --name django_dev \
  grant_app_dev
```

**Ollama Service:**
```bash
docker run --network=django-ollama-service \
  -d --gpus=all \
  -v ollama:/root/.ollama \
  -p 11434:11434 \
  --name ollama \
  ollama/ollama
```

### 5. Configure Ollama Models

```bash
docker exec -it ollama bash
ollama pull all-minilm
ollama pull gemma:2b
exit
```

### 6. Launch Neo4j Database

```bash
docker run \
  --restart always \
  --publish=7474:7474 --publish=7687:7687 \
  --volume=$HOME/neo4j/plugins:/plugins \
  --env NEO4J_AUTH=neo4j/amboralabs \
  --env 'NEO4J_dbms_security_procedures_unrestricted=apoc.*' \
  --env NEO4JLABS_PLUGINS='["apoc"]' \
  neo4j:5.19.0
```

## Service Endpoints

- **Django (Production)**: https://localhost (HTTPS), http://localhost (HTTP)
- **Django (Development)**: https://localhost:4343 (HTTPS), http://localhost:8080 (HTTP)
- **Ollama API**: http://localhost:11434
- **Neo4j Browser**: http://localhost:7474
- **Neo4j Bolt**: bolt://localhost:7687

## Data Management

### Backup Application Data

```bash
# Backup database
docker cp django:/var/app/db.sqlite3 backend/

# Backup uploads
docker cp django:/var/app/uploads/ backend/

# Backup vector store
docker cp django:/var/app/chroma/ backend/
```

### Restore Application Data

```bash
# Restore database
docker cp backend/db.sqlite3 django:/var/app/

# Restore uploads
docker cp backend/uploads/ django:/var/app/

# Restore vector store
docker cp backend/chroma/ django:/var/app/
```

## Troubleshooting

### SSL Certificate Permissions

If you encounter certificate errors:

```bash
sudo chmod 644 cert.pem privkey.pem
```

### Stop All Containers

```bash
docker stop $(docker ps -a -q)
```

### "myapi_ table not found" Error

Comment out the uploads section in the `get` function of the affected view.

### Container Logs

```bash
# Django logs
docker logs django

# Ollama logs
docker logs ollama

# Neo4j logs
docker logs <neo4j-container-name>
```

## Development Notes

- Development environment uses non-standard ports (8080, 4343) to avoid conflicts
- Ollama requires GPU support for optimal performance
- Neo4j credentials: `neo4j/amboralabs` (change in production)
- APOC procedures are enabled for advanced graph operations

## Security Considerations

- Change default Neo4j credentials before production deployment
- Ensure SSL certificates are properly secured and up to date
- Review and restrict network access in production environments
- Keep all Docker images and dependencies updated

---

For questions or issues, please contact the development team.

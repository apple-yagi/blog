---
https://zenn.dev/apple_yagi/articles/1151051620b735
---

# ローカルでPostgreSQLのDockerコンテナを立ち上げる

**Author:** やなぎ (Yanagi)
**Published:** 2023/07/02

## タグ選び

The article recommends checking the latest PostgreSQL Docker image versions on Docker Hub. As of the writing (2023/07/02), version 15.3 is recommended as the stable version.

## 立ち上げる (Setting Up)

The author suggests using Docker Compose instead of `docker run` for easier container management. Here's the recommended `compose.yaml` configuration:

```yaml
volumes:
  pg-data:

services:
  postgres:
    container_name: postgres
    image: postgres:15.3
    volumes:
      - pg-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DATABASE=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_ROOT_PASSWORD=root
```

To start the container, run:

```bash
docker compose up -d
```

To verify the container is running:

```bash
docker compose ps
```

The article emphasizes using Docker Compose for its simplicity and ease of configuration compared to long `docker run` commands.

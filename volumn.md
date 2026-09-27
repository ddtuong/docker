# Docker Volumes & Mounts

Docker containers are **ephemeral**: a container can be stopped, removed, and recreated at any time. Data that must survive the container lifecycle should therefore be stored outside the container's writable layer.

Docker provides several ways to mount persistent or temporary storage into containers.

---

## 1. Basic Architecture

```text
                         DOCKER HOST
┌─────────────────────────────────────────────────────┐
│                                                     │
│   ┌──────────────────────┐                          │
│   │      Container       │                          │
│   │                      │                          │
│   │   Application        │                          │
│   │                      │                          │
│   │   /app               │                          │
│   │   /data ◄────────────┼───────┐                  │
│   └──────────────────────┘       │                  │
│                                  │ mount            │
│                                  ▼                  │
│                         ┌─────────────────┐          │
│                         │     Storage     │          │
│                         │                 │          │
│                         │ Volume / Host   │          │
│                         │ directory / RAM │          │
│                         └─────────────────┘          │
│                                                     │
└─────────────────────────────────────────────────────┘
```

A **mount** connects external storage to a directory inside the container.

Example:

```text
my-data ───────────────► /app/data
  │                         │
  │                         │
  └── storage               └── mount point
```

The application only sees `/app/data`; it does not need to know where the data is physically stored.

---

# 2. Why Volumes?

Without a volume:

```text
Container
┌────────────────────┐
│ Application        │
│                    │
│ /data              │
│ └── database.db    │
└────────────────────┘
         │
         │ docker rm
         ▼
       ❌ Data lost
```

With a volume:

```text
Volume
┌────────────────────┐
│ database.db        │
└─────────┬──────────┘
          │ mount
          ▼
Container
┌────────────────────┐
│ /data              │
│ └── database.db    │
└────────────────────┘

docker rm
    │
    ▼
Container ❌

Volume
    │
    ▼
Data remains ✓
```

The key idea:

```text
Container = Application / Runtime
Volume    = Persistent Data
```

---

# 3. Types of Mounts

Docker commonly uses:

```text
                 Docker Storage
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
       Volume      Bind Mount     tmpfs
          │            │            │
    Docker manages   Host path     RAM
          │            │            │
     Persistent    Persistent    Temporary
```

There is also an **anonymous volume**, which is similar to a named volume but Docker generates the volume name automatically.

---

# 4. Named Volumes

A named volume is managed by Docker.

## Create a volume

```bash
docker volume create my-data
```

List volumes:

```bash
docker volume ls
```

Inspect a volume:

```bash
docker volume inspect my-data
```

## Mount the volume

```bash
docker run -d \
  --name my-app \
  -v my-data:/app/data \
  nginx
```

Architecture:

```text
Docker Host
│
├── Docker Volume
│   └── my-data
│
│       │ mount
│       ▼
│
└── Container
    └── /app/data
```

The syntax is:

```text
-v <volume-name>:<container-path>
```

Example:

```text
-v my-data:/app/data
   │        │
   │        └── path inside container
   └────────── volume name
```

## Remove the container

```bash
docker rm -f my-app
```

The container disappears, but:

```text
my-data
   │
   └── data still exists ✓
```

Create another container:

```bash
docker run -d \
  --name my-app-2 \
  -v my-data:/app/data \
  nginx
```

The new container can access the existing data.

### Typical use cases

Named volumes are commonly used for:

* MySQL
* PostgreSQL
* Redis
* Elasticsearch
* Application persistent data

---

# 5. Bind Mounts

A bind mount maps a specific directory on the host directly into the container.

```bash
docker run -d \
  -v ./data:/app/data \
  nginx
```

Architecture:

```text
HOST
┌──────────────────────┐
│ ./data/              │
│ ├── file1.txt        │
│ └── file2.txt        │
└──────────┬───────────┘
           │
           │ bind mount
           ▼
CONTAINER
┌──────────────────────┐
│ /app/data/           │
│ ├── file1.txt        │
│ └── file2.txt        │
└──────────────────────┘
```

Syntax:

```text
-v <host-path>:<container-path>
```

For example:

```bash
docker run -v C:\project\data:/app/data nginx
```

The same files are visible from:

```text
Host:
C:\project\data

Container:
/app/data
```

If the container creates:

```bash
echo "hello" > /app/data/test.txt
```

the host will also see:

```text
C:\project\data\test.txt
```

### Typical use cases

Bind mounts are useful for:

* Development
* Source code
* Configuration files
* Local datasets
* Sharing files between host and container

Example:

```bash
docker run \
  -v ./src:/app/src \
  my-python-app
```

Now code edited on the host is immediately visible inside the container.

---

# 6. Anonymous Volumes

An anonymous volume is created without specifying a name:

```bash
docker run \
  -v /app/data \
  nginx
```

Docker automatically generates a volume:

```text
Docker
└── Volumes
    └── 8f72c9a1...
          │
          ▼
      /app/data
```

Compare:

```bash
# Named volume
-v my-data:/app/data

# Anonymous volume
-v /app/data
```

Named volumes are usually easier to manage because you control their names.

---

# 7. tmpfs Mounts

`tmpfs` stores data in memory rather than persistent storage.

```bash
docker run \
  --tmpfs /app/tmp \
  nginx
```

Architecture:

```text
             HOST RAM
                │
                │
                ▼
        ┌───────────────┐
        │     tmpfs      │
        └───────┬───────┘
                │ mount
                ▼
        ┌────────────────┐
        │   Container    │
        │                │
        │   /app/tmp     │
        └────────────────┘
```

The data is temporary.

When the container disappears:

```text
Container ❌
     │
     ▼
tmpfs data ❌
```

Typical use cases:

* Temporary files
* Cache
* Short-lived data
* Data that should not be persisted to disk

---

# 8. `-v` vs `--mount`

Docker supports two syntaxes.

## `-v`

Short and convenient:

```bash
docker run \
  -v my-data:/app/data \
  nginx
```

## `--mount`

More explicit:

```bash
docker run \
  --mount type=volume,source=my-data,target=/app/data \
  nginx
```

For bind mount:

```bash
docker run \
  --mount type=bind,source=./data,target=/app/data \
  nginx
```

The concepts are the same; `--mount` makes the configuration more explicit.

---

# 9. Important Mount Terms

Consider:

```bash
-v my-data:/app/data
```

```text
-v
│
├── my-data
│     │
│     └── Volume name / source
│
└── /app/data
      │
      └── Mount point / target
```

### Source

Where the data comes from:

```text
my-data
```

### Target

Where the data appears inside the container:

```text
/app/data
```

### Mount

The relationship between them:

```text
my-data ─────────► /app/data
 source             target
```

---

# 10. Real Example: MySQL

Create a persistent volume:

```bash
docker volume create mysql-data
```

Run MySQL:

```bash
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -v mysql-data:/var/lib/mysql \
  mysql
```

Architecture:

```text
                     Docker Host
┌─────────────────────────────────────────────┐
│                                             │
│   Volume                                    │
│   ┌──────────────────────┐                  │
│   │ mysql-data            │                  │
│   │                       │                  │
│   │ MySQL database files  │                  │
│   └──────────┬───────────┘                  │
│              │                              │
│              │ mount                        │
│              ▼                              │
│   ┌──────────────────────────────┐          │
│   │          MySQL               │          │
│   │         Container            │          │
│   │                              │          │
│   │ /var/lib/mysql ◄─────────────┤          │
│   │                              │          │
│   └──────────────────────────────┘          │
│                                             │
└─────────────────────────────────────────────┘
```

If the MySQL container is removed:

```bash
docker rm -f mysql
```

the volume remains:

```text
mysql-data
    │
    └── Database data ✓
```

A new MySQL container can reuse it.

---

# 11. Commands Cheat Sheet

## Volumes

```bash
# Create
docker volume create my-data

# List
docker volume ls

# Inspect
docker volume inspect my-data

# Remove
docker volume rm my-data

# Remove unused volumes
docker volume prune
```

## Container with volume

```bash
docker run \
  -v my-data:/app/data \
  myimage
```

## Bind mount

```bash
docker run \
  -v ./data:/app/data \
  myimage
```

## tmpfs

```bash
docker run \
  --tmpfs /app/tmp \
  myimage
```

---

# 12. Quick Comparison

| Type             | Storage managed by | Persistent | Typical use                     |
| ---------------- | ------------------ | ---------: | ------------------------------- |
| Named Volume     | Docker             |        Yes | Database, application data      |
| Anonymous Volume | Docker             |        Yes | Temporary/automatic persistence |
| Bind Mount       | User/Host          |        Yes | Development, source code        |
| tmpfs            | RAM                |         No | Temporary data/cache            |

---

# 13. Mental Model

Remember this architecture:

```text
                    DOCKER
                       │
          ┌────────────┴────────────┐
          │                         │
      CONTAINER                  STORAGE
          │                         │
    Application              ┌──────┼──────┐
          │                   │      │      │
          │                Volume  Bind   tmpfs
          │                   │    Mount    │
          │                   │      │      │
          └──────────── mount ┴──────┴──────┘
```

The most important rule:

```text
Container
    ↓
Ephemeral / replaceable

Volume
    ↓
Persistent / independent
```

Therefore:

```text
❌ Don't rely on container filesystem for important data.

✓ Store persistent data in volumes.

✓ Use bind mounts when you specifically need
  to share host files/directories.

✓ Use tmpfs when the data only needs to exist
  temporarily in memory.
```

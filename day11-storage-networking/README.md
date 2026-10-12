# Day 11 — Docker Storage & Networking

## Objective

Deploy PostgreSQL in Docker with persistent storage, verify data persistence after container recreation, test container networking and DNS, troubleshoot a host-port conflict, and create a compressed volume backup.

## Environment

- Docker version: 29.5.3
- Database image: `postgres:16-alpine`
- Database container: `db`
- Database: `shop`
- Named volume: `pgdata`
- Custom bridge network: `app-net`

## 1. Named Volume and PostgreSQL

Created a named volume and a custom network:

```bash
docker network create app-net
docker volume create pgdata
```

Started PostgreSQL with the named volume mounted at `/var/lib/postgresql/data`.

The database was ready, as confirmed by:

```bash
docker exec db pg_isready -U postgres -d shop
```

Result:

```text
/var/run/postgresql:5432 - accepting connections
```

## 2. Data Persistence

Created a `notes` table and inserted the message `I survive`.

After removing the `db` container and recreating it with the same `pgdata` volume, the query still returned:

| id | message |
|---:|---|
| 1 | I survive |

This demonstrated that the database data persisted independently of the original container.

## 3. Container Networking and DNS

Connected to PostgreSQL from a separate temporary container using:

```bash
docker run --rm -it \
  --network app-net \
  -e PGPASSWORD=changeme \
  postgres:16-alpine \
  psql -h db -U postgres -d shop \
  -c "SELECT * FROM notes;"
```

The query successfully returned the saved row.

DNS comparison:

- Default bridge: `ping db` failed with `ping: bad address 'db'`.
- Custom bridge: `ping db` resolved to `172.19.0.2` and succeeded.
- A separate PostgreSQL client also successfully executed `SELECT 1;` through `app-net`.

These results demonstrated container-name resolution and database connectivity on the user-defined bridge network.

## 4. Port Mapping Troubleshooting

Started an Nginx container with:

```bash
docker run -d --name w1 -p 18080:80 nginx:alpine
```

`curl -I http://localhost:18080` returned `HTTP/1.1 200 OK`.

A second container attempted to publish the same host port and failed with:

```text
Bind for 0.0.0.0:18080 failed: port is already allocated
```

Removed the failed `w2` container and recreated it using host port `18081`:

```bash
docker rm -f w2
docker run -d --name w2 -p 18081:80 nginx:alpine
```

`curl -I http://localhost:18081` returned `HTTP/1.1 200 OK`.

## 5. Volume Backup

Stopped PostgreSQL before creating a file-level archive of the volume.

Backup file:

```text
pgdata-backup.tgz
```

Observed archive size: **6.5 MB**.

Verified that the archive contains PostgreSQL data files using:

```bash
tar -tzf pgdata-backup.tgz | head -20
```

Restarted PostgreSQL and confirmed it was accepting connections.

The archive was inspected, but a restore test has not yet been performed. A production backup process should include a tested restoration procedure.

## 6. Key Learnings

- Named volumes keep data separate from a container's writable layer.
- Recreating a container with the same volume allows existing database data to be reused.
- User-defined bridge networks support container-name DNS resolution.
- Two containers cannot simultaneously publish the same host IP/port combination.
- A volume is live storage; a backup is a separate copy intended for recovery.
- Database file-level backups should be created with the database stopped or by using an appropriate database-aware backup method.

## 7. Evidence

Store screenshots of actual terminal outputs in the `evidence/` directory:

1. Network and volume creation or inspection
2. PostgreSQL running and ready
3. Data before container recreation
4. Data after container recreation
5. Successful query from a separate container
6. Default bridge DNS failure and custom bridge DNS success
7. Port conflict and successful use of port `18081`
8. Backup archive size and PostgreSQL readiness after restart

## Conclusion

The lab demonstrated PostgreSQL data persistence using a Docker named volume, container communication through a user-defined bridge network, DNS-based service discovery, host-port conflict resolution, and creation of a compressed volume archive.

PostgreSQL Database for Hakaton Project
This repository contains a docker-compose.yml configuration for running a PostgreSQL database with automatic backups on deployment. The database is designed for the Hakaton project.
Overview
The setup includes:

A PostgreSQL database running in a Docker container.
Automatic backup creation whenever the database container is redeployed.
Persistent storage for database data and backups using Docker volumes.
A dedicated service for creating backups on deployment.

Database Configuration

Database Name: hakaton_db
User: postgres
Password: 159357
Host: hakaton_db-postgres-db-1
Port: 5434 (mapped to 5432 inside the container)
Connection URL: postgres://postgres:159357@hakaton_db-postgres-db-1:5434/hakaton_db

Prerequisites

Docker installed on your system.
Docker Compose installed.

Getting Started
1. Clone the Repository
git clone <repository-url>
cd <repository-directory>

2. Start the Database
Run the following command to start the PostgreSQL database and backup service:
docker-compose up -d

This will:

Start the postgres-db service, running PostgreSQL.
Start the backup-on-deploy service, which creates a backup of the database after the database is ready.

3. Verify the Setup

Check the logs of the PostgreSQL service to ensure it started correctly:
docker logs hakaton_db-postgres-db-1

Look for database system is ready to accept connections.

Check the logs of the backup service to confirm the backup was created:
docker logs hakaton_db-backup-on-deploy

Look for PostgreSQL is ready and no error messages.


4. Access the Database
You can connect to the database using a PostgreSQL client (e.g., psql, DBeaver, or pgAdmin) with the following details:

Host: localhost (or hakaton_db-postgres-db-1 inside the Docker network)
Port: 5434
Database: hakaton_db
User: postgres
Password: 159357

Example using psql:
psql -h localhost -p 5434 -U postgres -d hakaton_db

Backup Management
Automatic Backups

A backup is created automatically every time the postgres-db service is redeployed (e.g., when running docker-compose up -d).
Backups are stored in the backups Docker volume with filenames like backup_predeploy_YYYYMMDD_HHMMSS.sql.

To locate backups on the host:
docker volume inspect <project_name>_backups

Replace <project_name> with the name of the directory containing docker-compose.yml.
Manual Backups
To create a backup manually (e.g., after schema changes):
docker exec hakaton_db-postgres-db-1 bash -c "pg_dump -h localhost -U postgres hakaton_db > /backups/backup_manual_$(date +%Y%m%d_%H%M%S).sql"

Cleaning Old Backups
To prevent the backups volume from growing indefinitely, periodically remove backups older than 7 days:
docker exec hakaton_db-postgres-db-1 bash -c "find /backups -type f -mtime +7 -delete"

Restoring a Backup
To restore the database from a backup file:

Copy the backup file to a location accessible by the host.
Run:docker exec -i hakaton_db-postgres-db-1 psql -U postgres -d hakaton_db < /path/to/backup_file.sql



Stopping the Database
To stop and remove the containers:
docker-compose down

To also remove the database data and backups:
docker-compose down -v

Warning: The -v flag deletes the postgres_data and backups volumes, permanently removing all data and backups.
Troubleshooting
PostgreSQL Fails to Start

Check logs:docker logs hakaton_db-postgres-db-1


Ensure port 5434 is not in use on the host.
Verify the postgres_data volume is not corrupted:docker volume rm <project_name>_postgres_data
docker-compose up -d



Backup Service Fails

Check logs:docker logs hakaton_db-backup-on-deploy


If you see Error: PostgreSQL is not ready after 60 seconds, increase the timeout in the backup-on-deploy service's entrypoint (change {1..60} to {1..120}).

Connection Refused

Ensure the postgres-db service is running:docker ps


Verify the connection URL and credentials.

Notes

The backup-on-deploy service runs once per deployment and exits after creating a backup. It will restart on the next docker-compose up.
For integration with migration tools (e.g., Alembic or Flyway), run the manual backup command after applying migrations.
To customize backup retention or add scheduled backups, modify the docker-compose.yml or add a cron-based service.

License
This project is licensed under the MIT License.

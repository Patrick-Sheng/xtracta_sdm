# Awesome Project - Database Migration with SDM

## Environment Details

- **MySQL Version**: 5.7
- **SDM Version**: 0.6.4
- **Operating System**: Windows 
- **Docker Version**: 28.4.0

## How SDM Works

SDM (Schema Data Migration) is a database migration management tool that helps track and apply changes to MySQL databases in a controlled way.

It works by:
1. **Declarative Schema Management**: You define what you want your database tables to look like in SQL files (in the `schema/` folder), and SDM figures out what changes need to be made to get there.

2. **Migration Plans**: All changes are stored as JSON files in the `migration_plan/` folder. Each migration has a version number, a name, and dependencies on previous migrations, forming a clear chain of changes.

3. **Two Types of Migrations**:
   - **Schema migrations**: For structural changes (creating/altering tables, adding columns, etc.)
   - **Data migrations**: For data changes (inserting, updating, or deleting records)

4. **Migration Tracking**: SDM creates two tables (`_migration_history` and `_migration_history_log`) to track which migrations have been applied and maintain a complete audit log of all operations.

5. **Environment Management**: You can manage multiple database environments (production, dev, staging) with different configurations, applying the same migrations to each.

## Challenge Encountered and Solution

**Challenge**: When trying to run SDM commands through Docker on Windows, I encountered connection issues with `host.docker.internal` not working properly to connect the SDM container to my MySQL containers.

**Solution**: I created a custom Docker network (`sdm-network`) and connected all containers (mysql-prod, mysql-dev, and the SDM container) to this network. This allowed the containers to communicate using their container names as hostnames (e.g., `mysql-prod` instead of `127.0.0.1`). 

I also had to adjust the port configuration. Initially, I mapped the MySQL containers to external ports 3316 and 3317 (instead of the standard 3306) because port 3306 was already occupied by another MySQL instance running on my local machine. However, when containers communicate within the same Docker network, they use the internal container ports rather than the host-mapped ports. Therefore, I had to specify port 3306 in the SDM commands (the internal MySQL port) instead of the external mapped ports (3316/3317), as the SDM container connects directly to the MySQL containers through the Docker network rather than through the host machine.

Additionally, on Windows PowerShell, I had to adapt the Docker command syntax by:
- Removing the `-u $(id -u):$(id -g)` flag (not needed on Windows)
- Changing `$(pwd)` to `${PWD}` for the volume mount
- Using the `--network` flag to specify the Docker network
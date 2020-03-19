### Advanced Liquibase Migrations SQL Only

##### Concept
This is a port of the advanced-liquibase-migrations project (found at https://github.com/hdeiner/advanced-liquibase-migrations) to Liquibase. The only substinative difference is that rather than use the cross platform Liquibase changesets, the changesets consist only of non-portable SQL statements. 

We need to start treating our databases like code.  

- Keeping databases in code ensures that the process is well defined, automated, and completely transparent.  No more hidden work residing with specialized silos.
- Databases change with the code as part of emerengent design.  We should embrace that rather than deny it.
- Pets vs cattle analogy.  Would you treat your code like a pet and painstakingly patch each byte that changes from release to release hoping that everything still works, or want to be able to build, test, and then deploy it confidently from source using a version controlled repository?
- DevOps culture embraces the team as a whole creating outcomes, rather than exhaustive estimating, managing cross silo dependencies, and tracking work progress in what (by other names) is nothing different than a project plan.

This project demonstrates the use of an open source tool called Liquibase to help us create and migrate databases through their development and growth.  But what this project also demonstrates is that where SQL alone is inadequate to express a mrigration from version N to version N+1, we can use some of the more advanced features of Liquibase (such as a JDBC based migration instead if the usual SQL based migration).  In other words, this project shows how both Schema Migration and Data Migration can work together in a repository based manner with automated testing used to validate all of the actions.

When comparing FlyWay to Liquibase, one thing that distinguishes the two is the theoretical foundation of Liquibase.  It is the realization of the 2006 book by Ambler and Sadalage titled "Refactoring Databases: Evolutionary Database Design."  I consider Liquibase to be the first DevOps tool (way before the name became coined!), writtten by Nathan Voxland around the time that the book was first being read in the community.  Nathan sold Liquibase to Datical several years ago, but both he and the Datical company continue to exhance and support the tool to this day.

We will start with a database used by IMDB.  We will go through the following migrations:

- V1_1: the initial Schema.  This is a SQL based migration.
- V1_2: the initial static data.  This is a JDBC based migraion, because the data is only made available in TSV (tab seperated values) files.
- V2_1: a migration to add columns FIRST_NAME and LAST_NAME into the NAME table.  This is a SQL based migration.
- V2_2: a migration to split the PRIMARY_NAME column in the NAME table into the FIRST_NAME and LAST_NAME columns.  This is a JDBC based migration, because the steps to split are generally too difficult for SQL expression.
- V2_3: a migration to drop the PRIMARY_NAME column from the NAME table.  This is a SQL based migration.

##### Demo
###### step_1_start_mysql
When run, this script
```bash
#!/usr/bin/env bash

figlet -w 200 -f standard "Start MySQL in docker-composed Environment"

docker-compose -f docker-compose-mysql-and-mysql-data.yml up -d

figlet -w 160 -f small "Wait for MySQL to Start"
while true ; do
  result=$(docker logs mysql 2>&1 | grep -c "Version: '5.7.28'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)")
  if [ $result != 0 ] ; then
    echo "MySQL has started"
    break
  fi
  sleep 5
done

cd src/test/python
behave -v features/step_1_tests.feature
cd -
```
produces
```console
 ___  _             _     __  __       ____   ___  _       _             _            _                                                               _ 
/ ___|| |_ __ _ _ __| |_  |  \/  |_   _/ ___| / _ \| |     (_)_ __     __| | ___   ___| | _____ _ __       ___ ___  _ __ ___  _ __   ___  ___  ___  __| |
\___ \| __/ _` | '__| __| | |\/| | | | \___ \| | | | |     | | '_ \   / _` |/ _ \ / __| |/ / _ \ '__|____ / __/ _ \| '_ ` _ \| '_ \ / _ \/ __|/ _ \/ _` |
 ___) | || (_| | |  | |_  | |  | | |_| |___) | |_| | |___  | | | | | | (_| | (_) | (__|   <  __/ | |_____| (_| (_) | | | | | | |_) | (_) \__ \  __/ (_| |
|____/ \__\__,_|_|   \__| |_|  |_|\__, |____/ \__\_\_____| |_|_| |_|  \__,_|\___/ \___|_|\_\___|_|        \___\___/|_| |_| |_| .__/ \___/|___/\___|\__,_|
                                  |___/                                                                                      |_|                         
 _____            _                                      _   
| ____|_ ____   _(_)_ __ ___  _ __  _ __ ___   ___ _ __ | |_ 
|  _| | '_ \ \ / / | '__/ _ \| '_ \| '_ ` _ \ / _ \ '_ \| __|
| |___| | | \ V /| | | | (_) | | | | | | | | |  __/ | | | |_ 
|_____|_| |_|\_/ |_|_|  \___/|_| |_|_| |_| |_|\___|_| |_|\__|
                                                             
WARNING: The Docker Engine you're using is running in swarm mode.

Compose does not use swarm mode to deploy services to multiple nodes in a swarm. All containers will be scheduled on the current node.

To deploy your application across the swarm, use `docker stack deploy`.

Creating network "advanced-liquibase-migrations-sql-only_default" with the default driver
Creating mysql ... done
__      __    _ _      __           __  __      ___  ___  _      _         ___ _            _   
\ \    / /_ _(_) |_   / _|___ _ _  |  \/  |_  _/ __|/ _ \| |    | |_ ___  / __| |_ __ _ _ _| |_ 
 \ \/\/ / _` | |  _| |  _/ _ \ '_| | |\/| | || \__ \ (_) | |__  |  _/ _ \ \__ \  _/ _` | '_|  _|
  \_/\_/\__,_|_|\__| |_| \___/_|   |_|  |_|\_, |___/\__\_\____|  \__\___/ |___/\__\__,_|_|  \__|
                                           |__/                                                 
MySQL has started
Using defaults:
 stderr_capture True
          stage None
        dry_run False
   show_skipped True
 logging_format %(levelname)s:%(name)s:%(message)s
    log_capture True
        summary True
 default_format pretty
scenario_outline_annotation_schema {name} -- @{row.id} {examples.name}
          color True
  show_snippets True
          junit False
   default_tags 
       userdata {}
  steps_catalog False
 stdout_capture True
   show_timings True
    show_source True
  logging_level 20
Supplied path: "features/step_1_tests.feature"
Primary path is to a file so using its directory
Trying base directory: /home/howarddeiner/IdeaProjects/advanced-liquibase-migrations-sql-only/src/test/python/features
Feature: Start MySQL # features/step_1_tests.feature:1

  Scenario: Ensure that proper users are in database to allow Liquibase migrations  # features/step_1_tests.feature:3
    Given the database has no IMDB_Schema                                           # features/steps/step_1_tests.py:5 0.000s
    Then the "User" column in the "mysql.user" table should be                      # features/steps/step_1_tests.py:9 0.004s
      | User          |
      | root          |
      | user          |
      | mysql.session |
      | mysql.sys     |
      | root          |

1 feature passed, 0 failed, 0 skipped
1 scenario passed, 0 failed, 0 skipped
2 steps passed, 0 failed, 0 skipped, 0 undefined
Took 0m0.004s
```
We have merely started up a Docker container with a local MySQL database.

###### step_2_liquibase_migrate_V1.sh
When run, this script
```bash
#!/usr/bin/env bash

figlet -w 200 -f standard "Liquibase migrate to V1 (create initial database)"

mvn compile

cd src/main/java/common/data
./runAt1PerCent.sh
cd -

figlet -w 160 -f small "Liquibase V1_1 (initial schema)"
cd src/main/resources/db/migration
python3 manage_dbchangelog.py -i dbchangelog.xml -v V1_1
cd -
mvn liquibase:update

figlet -w 160 -f small "Liquibase V1_2 (initial static data)"
cd src/main/resources/db/migration
python3 manage_dbchangelog.py -i dbchangelog.xml -v V1_2
cd -
mvn liquibase:update
mvn liquibase:dbDoc

cd src/test/python
behave -v features/step_2_tests.feature
cd -
```
produces

```console
 _     _             _ _                                _                 _         _         __     ___    __                   _         _       _ _   _       _ 
| |   (_) __ _ _   _(_) |__   __ _ ___  ___   _ __ ___ (_) __ _ _ __ __ _| |_ ___  | |_ ___   \ \   / / |  / /___ _ __ ___  __ _| |_ ___  (_)_ __ (_) |_(_) __ _| |
| |   | |/ _` | | | | | '_ \ / _` / __|/ _ \ | '_ ` _ \| |/ _` | '__/ _` | __/ _ \ | __/ _ \   \ \ / /| | | |/ __| '__/ _ \/ _` | __/ _ \ | | '_ \| | __| |/ _` | |
| |___| | (_| | |_| | | |_) | (_| \__ \  __/ | | | | | | | (_| | | | (_| | ||  __/ | || (_) |   \ V / | | | | (__| | |  __/ (_| | ||  __/ | | | | | | |_| | (_| | |
|_____|_|\__, |\__,_|_|_.__/ \__,_|___/\___| |_| |_| |_|_|\__, |_|  \__,_|\__\___|  \__\___/     \_/  |_| | |\___|_|  \___|\__,_|\__\___| |_|_| |_|_|\__|_|\__,_|_|
            |_|                                           |___/                                            \_\                                                     
     _       _        _                  __  
  __| | __ _| |_ __ _| |__   __ _ ___  __\ \ 
 / _` |/ _` | __/ _` | '_ \ / _` / __|/ _ \ |
| (_| | (_| | || (_| | |_) | (_| \__ \  __/ |
 \__,_|\__,_|\__\__,_|_.__/ \__,_|___/\___| |
                                         /_/ 
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building advanced-liquibase-migrations-sql-only 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ advanced-liquibase-migrations-sql-only ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 7 resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ advanced-liquibase-migrations-sql-only ---
[INFO] Nothing to compile - all classes are up to date
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.146 s
[INFO] Finished at: 2020-03-19T14:06:12-04:00
[INFO] Final Memory: 12M/323M
[INFO] ------------------------------------------------------------------------
/home/howarddeiner/IdeaProjects/advanced-liquibase-migrations-sql-only
 _    _           _ _                   __   ___   _    ___      _ _   _      _          _                 __  
| |  (_)__ _ _  _(_) |__  __ _ ___ ___  \ \ / / | / |  / (_)_ _ (_) |_(_)__ _| |  ___ __| |_  ___ _ __  __ \ \ 
| |__| / _` | || | | '_ \/ _` (_-</ -_)  \ V /| | | | | || | ' \| |  _| / _` | | (_-</ _| ' \/ -_) '  \/ _` | |
|____|_\__, |\_,_|_|_.__/\__,_/__/\___|   \_/ |_|_|_| | ||_|_||_|_|\__|_\__,_|_| /__/\__|_||_\___|_|_|_\__,_| |
          |_|                                  |___|   \_\                                                 /_/ 
/home/howarddeiner/IdeaProjects/advanced-liquibase-migrations-sql-only
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building advanced-liquibase-migrations-sql-only 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- liquibase-maven-plugin:3.8.7:update (default-cli) @ advanced-liquibase-migrations-sql-only ---
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] 
[INFO] Liquibase Community 3.8.7 by Datical
[INFO] Starting Liquibase at Thu, 19 Mar 2020 14:06:14 EDT (version 3.8.7 #55 built at Mon Feb 24 03:04:51 UTC 2020)
[INFO] Executing on Database: jdbc:mysql://localhost:3306/zipster?useSSL=false
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOGLOCK
[INFO] CREATE TABLE DATABASECHANGELOGLOCK (ID INT NOT NULL, `LOCKED` BIT(1) NOT NULL, LOCKGRANTED datetime NULL, LOCKEDBY VARCHAR(255) NULL, CONSTRAINT PK_DATABASECHANGELOGLOCK PRIMARY KEY (ID))
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOGLOCK
[INFO] DELETE FROM DATABASECHANGELOGLOCK
[INFO] INSERT INTO DATABASECHANGELOGLOCK (ID, `LOCKED`) VALUES (1, 0)
[INFO] SELECT `LOCKED` FROM DATABASECHANGELOGLOCK WHERE ID=1
[INFO] Successfully acquired change log lock
[INFO] Creating database history table with name: DATABASECHANGELOG
[INFO] CREATE TABLE DATABASECHANGELOG (ID VARCHAR(255) NOT NULL, AUTHOR VARCHAR(255) NOT NULL, FILENAME VARCHAR(255) NOT NULL, DATEEXECUTED datetime NOT NULL, ORDEREXECUTED INT NOT NULL, EXECTYPE VARCHAR(10) NOT NULL, MD5SUM VARCHAR(35) NULL, `DESCRIPTION` VARCHAR(255) NULL, COMMENTS VARCHAR(255) NULL, TAG VARCHAR(255) NULL, LIQUIBASE VARCHAR(20) NULL, CONTEXTS VARCHAR(255) NULL, LABELS VARCHAR(255) NULL, DEPLOYMENT_ID VARCHAR(10) NULL)
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOG
[INFO] Reading from DATABASECHANGELOG
[INFO] SELECT * FROM DATABASECHANGELOG ORDER BY DATEEXECUTED ASC, ORDEREXECUTED ASC
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOGLOCK
[INFO] CREATE TABLE NAME (
                      NCONST            VARCHAR(31),
                      PRIMARY_NAME      VARCHAR(255),
                      BIRTH_YEAR        INT,
                      DEATH_YEAR        INT )
[INFO] CREATE TABLE TITLE (
                       TCONST            VARCHAR(31),
                       TITLE_TYPE        VARCHAR(31),
                       PRIMARY_TITLE     VARCHAR(1023),
                       ORIGINAL_TITLE    VARCHAR(1023),
                       IS_ADULT          bit,
                       START_YEAR        INT,
                       END_YEAR          INT,
                       RUNTIME_MINUTES   INT )
[INFO] CREATE TABLE NAME_PROFESSION (
                                 NCONST            VARCHAR(31),
                                 PROFESSION        VARCHAR(31) )
[INFO] CREATE TABLE NAME_TITLE (
                            NCONST            VARCHAR(31),
                            TCONST            VARCHAR(31) )
[INFO] CREATE TABLE TITLE_GENRE (
                             TCONST            VARCHAR(31),
                             GENRE             VARCHAR(31) )
[INFO] CREATE TABLE TITLE_AKA (
                           TCONST            VARCHAR(31),
                           ORDERING          INT,
                           TITLE             VARCHAR(1023),
                           REGION            VARCHAR(15),
                           LANGUAGE          VARCHAR(63),
                           TYPES             VARCHAR(63),
                           ATTRIBUTES        VARCHAR(63),
                           IS_ORIGINAL_TITLE bit )
[INFO] CREATE TABLE TITLE_DIRECTOR (
                                TCONST            VARCHAR(31),
                                NCONST            VARCHAR(31) )
[INFO] CREATE TABLE TITLE_WRITER (
                              TCONST            VARCHAR(31),
                              NCONST            VARCHAR(31) )
[INFO] CREATE TABLE TITLE_EPISODE (
                               TCONST            VARCHAR(31),
                               TCONST_PARENT     VARCHAR(31),
                               SEASON_NUMBER     INT,
                               EPISODE_NUMBER    INT )
[INFO] CREATE TABLE TITLE_PRINCIPALS (
                                  TCONST            VARCHAR(31),
                                  ORDERING          INT,
                                  NCONST            VARCHAR(31),
                                  CATEGORY          VARCHAR(63),
                                  JOB               VARCHAR(255),
                                  CHARACTER_PLAYED  VARCHAR(255) )
[INFO] CREATE TABLE TITLE_RATING (
                              TCONST            VARCHAR(31),
                              AVERAGE_RATING    VARCHAR(15),
                              NUMBER_OF_VOTES   INT )
[INFO] Custom SQL executed
[INFO] ChangeSet src/main/resources/db/migration/V1_1__Create_Initial_IMDB_Schema.xml::V1_1 Create Initial IMBD Schema::howarddeiner ran successfully in 724ms
[INFO] SELECT MAX(ORDEREXECUTED) FROM DATABASECHANGELOG
[INFO] INSERT INTO DATABASECHANGELOG (ID, AUTHOR, FILENAME, DATEEXECUTED, ORDEREXECUTED, MD5SUM, `DESCRIPTION`, COMMENTS, EXECTYPE, CONTEXTS, LABELS, LIQUIBASE, DEPLOYMENT_ID) VALUES ('V1_1 Create Initial IMBD Schema', 'howarddeiner', 'src/main/resources/db/migration/V1_1__Create_Initial_IMDB_Schema.xml', NOW(), 1, '8:19f5854b2dd8892e2f552911c6b59ecd', 'sql', '', 'EXECUTED', NULL, NULL, '3.8.7', '4641176130')
[INFO] Successfully released change log lock
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 3.519 s
[INFO] Finished at: 2020-03-19T14:06:16-04:00
[INFO] Final Memory: 17M/267M
[INFO] ------------------------------------------------------------------------
 _    _           _ _                   __   ___   ___    ___      _ _   _      _      _        _   _         _      _      __  
| |  (_)__ _ _  _(_) |__  __ _ ___ ___  \ \ / / | |_  )  / (_)_ _ (_) |_(_)__ _| |  __| |_ __ _| |_(_)__   __| |__ _| |_ __ \ \ 
| |__| / _` | || | | '_ \/ _` (_-</ -_)  \ V /| |  / /  | || | ' \| |  _| / _` | | (_-<  _/ _` |  _| / _| / _` / _` |  _/ _` | |
|____|_\__, |\_,_|_|_.__/\__,_/__/\___|   \_/ |_|_/___| | ||_|_||_|_|\__|_\__,_|_| /__/\__\__,_|\__|_\__| \__,_\__,_|\__\__,_| |
          |_|                                  |___|     \_\                                                                /_/ 
/home/howarddeiner/IdeaProjects/advanced-liquibase-migrations-sql-only
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building advanced-liquibase-migrations-sql-only 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- liquibase-maven-plugin:3.8.7:update (default-cli) @ advanced-liquibase-migrations-sql-only ---
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] 
[INFO] Liquibase Community 3.8.7 by Datical
[INFO] Starting Liquibase at Thu, 19 Mar 2020 14:06:20 EDT (version 3.8.7 #55 built at Mon Feb 24 03:04:51 UTC 2020)
[INFO] Executing on Database: jdbc:mysql://localhost:3306/zipster?useSSL=false
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOGLOCK
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOGLOCK
[INFO] SELECT `LOCKED` FROM DATABASECHANGELOGLOCK WHERE ID=1
[INFO] Successfully acquired change log lock
[INFO] SELECT MD5SUM FROM DATABASECHANGELOG WHERE MD5SUM IS NOT NULL LIMIT 1
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOG
[INFO] Reading from DATABASECHANGELOG
[INFO] SELECT * FROM DATABASECHANGELOG ORDER BY DATEEXECUTED ASC, ORDEREXECUTED ASC
src/main/java/common/data/name.basics.tsv.smaller - 6235 total lines (15 SECOND REPORTING INTERVAL) 
src/main/java/common/data/name.basics.tsv.smaller - 60% complete - elapsed time = 0:15 - remaining time = 0:09
src/main/java/common/data/name.basics.tsv.smaller - 100% complete - elapsed time = 0:23 - remaining time = 0:00
src/main/java/common/data/title.akas.tsv.smaller - 2573 total lines (15 SECOND REPORTING INTERVAL) 
src/main/java/common/data/title.akas.tsv.smaller - 100% complete - elapsed time = 0:01 - remaining time = 0:00
src/main/java/common/data/title.basics.tsv.smaller - 3736 total lines (15 SECOND REPORTING INTERVAL) 
src/main/java/common/data/title.basics.tsv.smaller - 100% complete - elapsed time = 0:05 - remaining time = 0:00
src/main/java/common/data/title.crew.tsv.smaller - 3737 total lines (15 SECOND REPORTING INTERVAL) 
src/main/java/common/data/title.crew.tsv.smaller - 100% complete - elapsed time = 0:03 - remaining time = 0:00
src/main/java/common/data/title.episode.tsv.smaller - 2541 total lines (15 SECOND REPORTING INTERVAL) 
src/main/java/common/data/title.episode.tsv.smaller - 100% complete - elapsed time = 0:01 - remaining time = 0:00
src/main/java/common/data/title.principals.tsv.smaller - 21256 total lines (15 SECOND REPORTING INTERVAL) 
src/main/java/common/data/title.principals.tsv.smaller - 100% complete - elapsed time = 0:10 - remaining time = 0:00
src/main/java/common/data/title.ratings.tsv.smaller - 613 total lines (15 SECOND REPORTING INTERVAL) 
src/main/java/common/data/title.ratings.tsv.smaller - 100% complete - elapsed time = 0:00 - remaining time = 0:00
[INFO] null
[INFO] ChangeSet src/main/resources/db/migration/V1_2__Load_Initial_IMDB_Data.xml::V1_2 Add First and Last Name Columns::howarddeiner ran successfully in 46465ms
[INFO] SELECT MAX(ORDEREXECUTED) FROM DATABASECHANGELOG
[INFO] INSERT INTO DATABASECHANGELOG (ID, AUTHOR, FILENAME, DATEEXECUTED, ORDEREXECUTED, MD5SUM, `DESCRIPTION`, COMMENTS, EXECTYPE, CONTEXTS, LABELS, LIQUIBASE, DEPLOYMENT_ID) VALUES ('V1_2 Add First and Last Name Columns', 'howarddeiner', 'src/main/resources/db/migration/V1_2__Load_Initial_IMDB_Data.xml', NOW(), 2, '8:2e6b71d7d07c6dc604b522ccc7d74fbb', 'customChange', '', 'EXECUTED', NULL, NULL, '3.8.7', '4641181610')
[INFO] Successfully released change log lock
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 49.211 s
[INFO] Finished at: 2020-03-19T14:07:08-04:00
[INFO] Final Memory: 17M/349M
[INFO] ------------------------------------------------------------------------
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building advanced-liquibase-migrations-sql-only 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- liquibase-maven-plugin:3.8.7:dbDoc (default-cli) @ advanced-liquibase-migrations-sql-only ---
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] 
[INFO] Liquibase Community 3.8.7 by Datical
[INFO] Starting Liquibase at Thu, 19 Mar 2020 14:07:10 EDT (version 3.8.7 #55 built at Mon Feb 24 03:04:51 UTC 2020)
[INFO] Executing on Database: jdbc:mysql://localhost:3306/zipster?useSSL=false
[INFO] Generating Database Documentation
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOGLOCK
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOGLOCK
[INFO] SELECT `LOCKED` FROM DATABASECHANGELOGLOCK WHERE ID=1
[INFO] Successfully acquired change log lock
[INFO] SELECT MD5SUM FROM DATABASECHANGELOG WHERE MD5SUM IS NOT NULL LIMIT 1
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOG
[INFO] Reading from DATABASECHANGELOG
[INFO] SELECT * FROM DATABASECHANGELOG ORDER BY DATEEXECUTED ASC, ORDEREXECUTED ASC
[INFO] Successfully released change log lock
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 3.331 s
[INFO] Finished at: 2020-03-19T14:07:12-04:00
[INFO] Final Memory: 17M/355M
[INFO] ------------------------------------------------------------------------
Using defaults:
        dry_run False
    log_capture True
  steps_catalog False
   show_timings True
 stderr_capture True
    show_source True
        summary True
scenario_outline_annotation_schema {name} -- @{row.id} {examples.name}
       userdata {}
 default_format pretty
 stdout_capture True
   show_skipped True
          junit False
   default_tags 
          color True
          stage None
  logging_level 20
  show_snippets True
 logging_format %(levelname)s:%(name)s:%(message)s
Supplied path: "features/step_2_tests.feature"
Primary path is to a file so using its directory
Trying base directory: /home/howarddeiner/IdeaProjects/advanced-liquibase-migrations-sql-only/src/test/python/features
Feature: Version 1 Database # features/step_2_tests.feature:1

  Scenario: Ensure that the V1_1 and V1_2 Liquibase migrations happened correctly  # features/step_2_tests.feature:3
    Given the database is at version "V1_2"                                        # features/steps/common_steps.py:6 0.000s
    Then the "zipster" database schema should be                                   # features/steps/common_steps.py:10
      | TABLE_NAME            | COLUMN_NAME       | ORDINAL_POSITION | DATA_TYPE | CHARACTER_MAXIMUM_LENGTH |
      | DATABASECHANGELOG     | ID                | 1                | varchar   | 255                      |
      | DATABASECHANGELOG     | AUTHOR            | 2                | varchar   | 255                      |
      | DATABASECHANGELOG     | FILENAME          | 3                | varchar   | 255                      |
      | DATABASECHANGELOG     | DATEEXECUTED      | 4                | datetime  | NULL                     |
      | DATABASECHANGELOG     | ORDEREXECUTED     | 5                | int       | NULL                     |
      | DATABASECHANGELOG     | EXECTYPE          | 6                | varchar   | 10                       |
      | DATABASECHANGELOG     | MD5SUM            | 7                | varchar   | 35                       |
      | DATABASECHANGELOG     | DESCRIPTION       | 8                | varchar   | 255                      |
      | DATABASECHANGELOG     | COMMENTS          | 9                | varchar   | 255                      |
      | DATABASECHANGELOG     | TAG               | 10               | varchar   | 255                      |
      | DATABASECHANGELOG     | LIQUIBASE         | 11               | varchar   | 20                       |
      | DATABASECHANGELOG     | CONTEXTS          | 12               | varchar   | 255                      |
      | DATABASECHANGELOG     | LABELS            | 13               | varchar   | 255                      |
    Then the "zipster" database schema should be                                   # features/steps/common_steps.py:10 0.005s
      | TABLE_NAME            | COLUMN_NAME       | ORDINAL_POSITION | DATA_TYPE | CHARACTER_MAXIMUM_LENGTH |
      | DATABASECHANGELOG     | ID                | 1                | varchar   | 255                      |
      | DATABASECHANGELOG     | AUTHOR            | 2                | varchar   | 255                      |
      | DATABASECHANGELOG     | FILENAME          | 3                | varchar   | 255                      |
      | DATABASECHANGELOG     | DATEEXECUTED      | 4                | datetime  | NULL                     |
      | DATABASECHANGELOG     | ORDEREXECUTED     | 5                | int       | NULL                     |
      | DATABASECHANGELOG     | EXECTYPE          | 6                | varchar   | 10                       |
      | DATABASECHANGELOG     | MD5SUM            | 7                | varchar   | 35                       |
      | DATABASECHANGELOG     | DESCRIPTION       | 8                | varchar   | 255                      |
      | DATABASECHANGELOG     | COMMENTS          | 9                | varchar   | 255                      |
      | DATABASECHANGELOG     | TAG               | 10               | varchar   | 255                      |
      | DATABASECHANGELOG     | LIQUIBASE         | 11               | varchar   | 20                       |
      | DATABASECHANGELOG     | CONTEXTS          | 12               | varchar   | 255                      |
      | DATABASECHANGELOG     | LABELS            | 13               | varchar   | 255                      |
      | DATABASECHANGELOG     | DEPLOYMENT_ID     | 14               | varchar   | 10                       |
      | DATABASECHANGELOGLOCK | ID                | 1                | int       | NULL                     |
      | DATABASECHANGELOGLOCK | LOCKED            | 2                | bit       | NULL                     |
      | DATABASECHANGELOGLOCK | LOCKGRANTED       | 3                | datetime  | NULL                     |
      | DATABASECHANGELOGLOCK | LOCKEDBY          | 4                | varchar   | 255                      |
      | NAME                  | NCONST            | 1                | varchar   | 31                       |
      | NAME                  | PRIMARY_NAME      | 2                | varchar   | 255                      |
      | NAME                  | BIRTH_YEAR        | 3                | int       | NULL                     |
      | NAME                  | DEATH_YEAR        | 4                | int       | NULL                     |
      | NAME_PROFESSION       | NCONST            | 1                | varchar   | 31                       |
      | NAME_PROFESSION       | PROFESSION        | 2                | varchar   | 31                       |
      | NAME_TITLE            | NCONST            | 1                | varchar   | 31                       |
      | NAME_TITLE            | TCONST            | 2                | varchar   | 31                       |
      | TITLE                 | TCONST            | 1                | varchar   | 31                       |
      | TITLE                 | TITLE_TYPE        | 2                | varchar   | 31                       |
      | TITLE                 | PRIMARY_TITLE     | 3                | varchar   | 1023                     |
      | TITLE                 | ORIGINAL_TITLE    | 4                | varchar   | 1023                     |
      | TITLE                 | IS_ADULT          | 5                | bit       | NULL                     |
      | TITLE                 | START_YEAR        | 6                | int       | NULL                     |
      | TITLE                 | END_YEAR          | 7                | int       | NULL                     |
      | TITLE                 | RUNTIME_MINUTES   | 8                | int       | NULL                     |
      | TITLE_AKA             | TCONST            | 1                | varchar   | 31                       |
      | TITLE_AKA             | ORDERING          | 2                | int       | NULL                     |
      | TITLE_AKA             | TITLE             | 3                | varchar   | 1023                     |
      | TITLE_AKA             | REGION            | 4                | varchar   | 15                       |
      | TITLE_AKA             | LANGUAGE          | 5                | varchar   | 63                       |
      | TITLE_AKA             | TYPES             | 6                | varchar   | 63                       |
      | TITLE_AKA             | ATTRIBUTES        | 7                | varchar   | 63                       |
      | TITLE_AKA             | IS_ORIGINAL_TITLE | 8                | bit       | NULL                     |
      | TITLE_DIRECTOR        | TCONST            | 1                | varchar   | 31                       |
      | TITLE_DIRECTOR        | NCONST            | 2                | varchar   | 31                       |
      | TITLE_EPISODE         | TCONST            | 1                | varchar   | 31                       |
      | TITLE_EPISODE         | TCONST_PARENT     | 2                | varchar   | 31                       |
      | TITLE_EPISODE         | SEASON_NUMBER     | 3                | int       | NULL                     |
      | TITLE_EPISODE         | EPISODE_NUMBER    | 4                | int       | NULL                     |
      | TITLE_GENRE           | TCONST            | 1                | varchar   | 31                       |
      | TITLE_GENRE           | GENRE             | 2                | varchar   | 31                       |
      | TITLE_PRINCIPALS      | TCONST            | 1                | varchar   | 31                       |
      | TITLE_PRINCIPALS      | ORDERING          | 2                | int       | NULL                     |
      | TITLE_PRINCIPALS      | NCONST            | 3                | varchar   | 31                       |
      | TITLE_PRINCIPALS      | CATEGORY          | 4                | varchar   | 63                       |
      | TITLE_PRINCIPALS      | JOB               | 5                | varchar   | 255                      |
      | TITLE_PRINCIPALS      | CHARACTER_PLAYED  | 6                | varchar   | 255                      |
      | TITLE_RATING          | TCONST            | 1                | varchar   | 31                       |
      | TITLE_RATING          | AVERAGE_RATING    | 2                | varchar   | 15                       |
      | TITLE_RATING          | NUMBER_OF_VOTES   | 3                | int       | NULL                     |
      | TITLE_WRITER          | TCONST            | 1                | varchar   | 31                       |
      | TITLE_WRITER          | NCONST            | 2                | varchar   | 31                       |
    And the following tables have the following row counts                         # features/steps/common_steps.py:38 0.031s
      | TABLE_NAME       | ROW_COUNT |
      | NAME             | 6234      |
      | NAME_PROFESSION  | 16300     |
      | NAME_TITLE       | 24389     |
      | TITLE            | 3735      |
      | TITLE_AKA        | 2572      |
      | TITLE_DIRECTOR   | 3620      |
      | TITLE_EPISODE    | 2540      |
      | TITLE_GENRE      | 7228      |
      | TITLE_PRINCIPALS | 21255     |
      | TITLE_RATING     | 612       |
      | TITLE_WRITER     | 2449      |

1 feature passed, 0 failed, 0 skipped
1 scenario passed, 0 failed, 0 skipped
3 steps passed, 0 failed, 0 skipped, 0 undefined
Took 0m0.036s
```

The script is a little tedious in showing everything going on in deep detail.

Reading along, the script first compiles all of the Java code, invokes a script to get rid of 90% of the data to import (to make the import time more bearable).

Because of the way that Liquibase is a much more rigorous tool in keeping with the idea of developing on trunk in a version controlled repository fashion, I had to fashion the dbchangelog.xml to allow us to see the effects of migrating from one database version to another, all within the confines of one project.  I did this by breaking out the versions of the database migration state in the same way that I did these migrations using Liquibase, and then writing a small Python script to allow me to comment out migrations that are beyond the migration state version that I want to stop at.

The script calls on the manage_dbchangelog.py script and calls on Liquibase to migrates to version 1_1 and then 1_2.

At the end of the script, one can use tools like MySQL Workbench to look around in the database created.  A nice automated BDD test is here to show us that we did what we said we would do!

###### step_3_liquibase_migrate_V2.sh
When run, this script
```bash
#!/usr/bin/env bash

figlet -w 200 -f standard "Liquibase migrate to V2 (split name)"

figlet -w 160 -f small "Liquibase V2_1 (add first and last name)"
cd src/main/resources/db/migration
python3 manage_dbchangelog.py -i dbchangelog.xml -v V2_1
cd -
mvn liquibase:update

figlet -w 160 -f small "Liquibase V2_2 (split primary_name into first and last name)"
cd src/main/resources/db/migration
python3 manage_dbchangelog.py -i dbchangelog.xml -v V2_2
cd -
mvn liquibase:update

figlet -w 160 -f small "Liquibase V2_3 (remove primary_name)"
cd src/main/resources/db/migration
python3 manage_dbchangelog.py -i dbchangelog.xml -v V2_3
cd -
mvn liquibase:update
mvn liquibase:dbDoc

cd src/test/python
behave -v features/step_3_tests.feature
cd -
```
produces

```console
 _     _             _ _                                _                 _         _         __     ______     __         _ _ _                              __  
 _     _             _ _                                _                 _         _         __     ______     __         _ _ _                              __  
| |   (_) __ _ _   _(_) |__   __ _ ___  ___   _ __ ___ (_) __ _ _ __ __ _| |_ ___  | |_ ___   \ \   / /___ \   / /__ _ __ | (_) |_   _ __   __ _ _ __ ___   __\ \ 
| |   | |/ _` | | | | | '_ \ / _` / __|/ _ \ | '_ ` _ \| |/ _` | '__/ _` | __/ _ \ | __/ _ \   \ \ / /  __) | | / __| '_ \| | | __| | '_ \ / _` | '_ ` _ \ / _ \ |
| |___| | (_| | |_| | | |_) | (_| \__ \  __/ | | | | | | | (_| | | | (_| | ||  __/ | || (_) |   \ V /  / __/  | \__ \ |_) | | | |_  | | | | (_| | | | | | |  __/ |
|_____|_|\__, |\__,_|_|_.__/ \__,_|___/\___| |_| |_| |_|_|\__, |_|  \__,_|\__\___|  \__\___/     \_/  |_____| | |___/ .__/|_|_|\__| |_| |_|\__,_|_| |_| |_|\___| |
            |_|                                           |___/                                                \_\  |_|                                       /_/ 
 _    _           _ _                   __   _____   _    __       _    _    __ _        _                  _   _         _                       __  
| |  (_)__ _ _  _(_) |__  __ _ ___ ___  \ \ / /_  ) / |  / /_ _ __| |__| |  / _(_)_ _ __| |_   __ _ _ _  __| | | |__ _ __| |_   _ _  __ _ _ __  __\ \ 
| |__| / _` | || | | '_ \/ _` (_-</ -_)  \ V / / /  | | | / _` / _` / _` | |  _| | '_(_-<  _| / _` | ' \/ _` | | / _` (_-<  _| | ' \/ _` | '  \/ -_) |
|____|_\__, |\_,_|_|_.__/\__,_/__/\___|   \_/ /___|_|_| | \__,_\__,_\__,_| |_| |_|_| /__/\__| \__,_|_||_\__,_| |_\__,_/__/\__| |_||_\__,_|_|_|_\___| |
          |_|                                    |___|   \_\                                                                                      /_/ 
/home/howarddeiner/IdeaProjects/advanced-liquibase-migrations-sql-only
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building advanced-liquibase-migrations-sql-only 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- liquibase-maven-plugin:3.8.7:update (default-cli) @ advanced-liquibase-migrations-sql-only ---
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] 
[INFO] Liquibase Community 3.8.7 by Datical
[INFO] Starting Liquibase at Thu, 19 Mar 2020 14:08:10 EDT (version 3.8.7 #55 built at Mon Feb 24 03:04:51 UTC 2020)
[INFO] Executing on Database: jdbc:mysql://localhost:3306/zipster?useSSL=false
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOGLOCK
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOGLOCK
[INFO] SELECT `LOCKED` FROM DATABASECHANGELOGLOCK WHERE ID=1
[INFO] Successfully acquired change log lock
[INFO] SELECT MD5SUM FROM DATABASECHANGELOG WHERE MD5SUM IS NOT NULL LIMIT 1
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOG
[INFO] Reading from DATABASECHANGELOG
[INFO] SELECT * FROM DATABASECHANGELOG ORDER BY DATEEXECUTED ASC, ORDEREXECUTED ASC
[INFO] ALTER TABLE NAME
            ADD COLUMN FIRST_NAME VARCHAR(255) AFTER PRIMARY_NAME,
            ADD COLUMN LAST_NAME VARCHAR(255) AFTER FIRST_NAME
[INFO] Custom SQL executed
[INFO] ChangeSet src/main/resources/db/migration/V2_1__Add_First_and_Last_Name_Columns.xml::V2_1 Add First and Last Name Columns::howarddeiner ran successfully in 628ms
[INFO] SELECT MAX(ORDEREXECUTED) FROM DATABASECHANGELOG
[INFO] INSERT INTO DATABASECHANGELOG (ID, AUTHOR, FILENAME, DATEEXECUTED, ORDEREXECUTED, MD5SUM, `DESCRIPTION`, COMMENTS, EXECTYPE, CONTEXTS, LABELS, LIQUIBASE, DEPLOYMENT_ID) VALUES ('V2_1 Add First and Last Name Columns', 'howarddeiner', 'src/main/resources/db/migration/V2_1__Add_First_and_Last_Name_Columns.xml', NOW(), 3, '8:8749ecad1a784e70745e5d7df086595e', 'sql', '', 'EXECUTED', NULL, NULL, '3.8.7', '4641291332')
[INFO] Successfully released change log lock
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2.795 s
[INFO] Finished at: 2020-03-19T14:08:11-04:00
[INFO] Final Memory: 17M/355M
[INFO] ------------------------------------------------------------------------
 _    _           _ _                   __   _____   ___    __       _ _ _              _                                             _     _       
| |  (_)__ _ _  _(_) |__  __ _ ___ ___  \ \ / /_  ) |_  )  / /___ __| (_) |_   _ __ _ _(_)_ __  __ _ _ _ _  _   _ _  __ _ _ __  ___  (_)_ _| |_ ___ 
| |__| / _` | || | | '_ \/ _` (_-</ -_)  \ V / / /   / /  | (_-< '_ \ | |  _| | '_ \ '_| | '  \/ _` | '_| || | | ' \/ _` | '  \/ -_) | | ' \  _/ _ \
|____|_\__, |\_,_|_|_.__/\__,_/__/\___|   \_/ /___|_/___| | /__/ .__/_|_|\__| | .__/_| |_|_|_|_\__,_|_|  \_, |_|_||_\__,_|_|_|_\___| |_|_||_\__\___/
          |_|                                    |___|     \_\ |_|            |_|                        |__/___|                                   
  __ _        _                  _   _         _                       __  
 / _(_)_ _ __| |_   __ _ _ _  __| | | |__ _ __| |_   _ _  __ _ _ __  __\ \ 
|  _| | '_(_-<  _| / _` | ' \/ _` | | / _` (_-<  _| | ' \/ _` | '  \/ -_) |
|_| |_|_| /__/\__| \__,_|_||_\__,_| |_\__,_/__/\__| |_||_\__,_|_|_|_\___| |
                                                                       /_/ 
/home/howarddeiner/IdeaProjects/advanced-liquibase-migrations-sql-only
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building advanced-liquibase-migrations-sql-only 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- liquibase-maven-plugin:3.8.7:update (default-cli) @ advanced-liquibase-migrations-sql-only ---
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] 
[INFO] Liquibase Community 3.8.7 by Datical
[INFO] Starting Liquibase at Thu, 19 Mar 2020 14:08:14 EDT (version 3.8.7 #55 built at Mon Feb 24 03:04:51 UTC 2020)
[INFO] Executing on Database: jdbc:mysql://localhost:3306/zipster?useSSL=false
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOGLOCK
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOGLOCK
[INFO] SELECT `LOCKED` FROM DATABASECHANGELOGLOCK WHERE ID=1
[INFO] Successfully acquired change log lock
[INFO] SELECT MD5SUM FROM DATABASECHANGELOG WHERE MD5SUM IS NOT NULL LIMIT 1
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOG
[INFO] Reading from DATABASECHANGELOG
[INFO] SELECT * FROM DATABASECHANGELOG ORDER BY DATEEXECUTED ASC, ORDEREXECUTED ASC
splitting PRIMARY_NAME - 6234 total lines (60 SECOND REPORTING INTERVAL) 
[INFO] null
[INFO] ChangeSet src/main/resources/db/migration/V2_2__Split_Primary_Name.xml::V2_2 Split Primary Name::howarddeiner ran successfully in 18873ms
[INFO] SELECT MAX(ORDEREXECUTED) FROM DATABASECHANGELOG
[INFO] INSERT INTO DATABASECHANGELOG (ID, AUTHOR, FILENAME, DATEEXECUTED, ORDEREXECUTED, MD5SUM, `DESCRIPTION`, COMMENTS, EXECTYPE, CONTEXTS, LABELS, LIQUIBASE, DEPLOYMENT_ID) VALUES ('V2_2 Split Primary Name', 'howarddeiner', 'src/main/resources/db/migration/V2_2__Split_Primary_Name.xml', NOW(), 4, '8:1b470067fb839279e2777f5be720df7d', 'customChange', '', 'EXECUTED', NULL, NULL, '3.8.7', '4641295758')
[INFO] Successfully released change log lock
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 21.378 s
[INFO] Finished at: 2020-03-19T14:08:34-04:00
[INFO] Final Memory: 17M/343M
[INFO] ------------------------------------------------------------------------
 _    _           _ _                   __   _____   ____   __                                    _                                         __  
| |  (_)__ _ _  _(_) |__  __ _ ___ ___  \ \ / /_  ) |__ /  / / _ ___ _ __  _____ _____   _ __ _ _(_)_ __  __ _ _ _ _  _   _ _  __ _ _ __  __\ \ 
| |__| / _` | || | | '_ \/ _` (_-</ -_)  \ V / / /   |_ \ | | '_/ -_) '  \/ _ \ V / -_) | '_ \ '_| | '  \/ _` | '_| || | | ' \/ _` | '  \/ -_) |
|____|_\__, |\_,_|_|_.__/\__,_/__/\___|   \_/ /___|_|___/ | |_| \___|_|_|_\___/\_/\___| | .__/_| |_|_|_|_\__,_|_|  \_, |_|_||_\__,_|_|_|_\___| |
          |_|                                    |___|     \_\                          |_|                        |__/___|                 /_/ 
/home/howarddeiner/IdeaProjects/advanced-liquibase-migrations-sql-only
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building advanced-liquibase-migrations-sql-only 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- liquibase-maven-plugin:3.8.7:update (default-cli) @ advanced-liquibase-migrations-sql-only ---
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] 
[INFO] Liquibase Community 3.8.7 by Datical
[INFO] Starting Liquibase at Thu, 19 Mar 2020 14:08:37 EDT (version 3.8.7 #55 built at Mon Feb 24 03:04:51 UTC 2020)
[INFO] Executing on Database: jdbc:mysql://localhost:3306/zipster?useSSL=false
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOGLOCK
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOGLOCK
[INFO] SELECT `LOCKED` FROM DATABASECHANGELOGLOCK WHERE ID=1
[INFO] Successfully acquired change log lock
[INFO] SELECT MD5SUM FROM DATABASECHANGELOG WHERE MD5SUM IS NOT NULL LIMIT 1
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOG
[INFO] Reading from DATABASECHANGELOG
[INFO] SELECT * FROM DATABASECHANGELOG ORDER BY DATEEXECUTED ASC, ORDEREXECUTED ASC
[INFO] ALTER TABLE NAME
            DROP COLUMN PRIMARY_NAME
[INFO] Custom SQL executed
[INFO] ChangeSet src/main/resources/db/migration/V2_3__Drop_Primary_Name_Column.xml::V2_3 Drop Primary Name Column::howarddeiner ran successfully in 613ms
[INFO] SELECT MAX(ORDEREXECUTED) FROM DATABASECHANGELOG
[INFO] INSERT INTO DATABASECHANGELOG (ID, AUTHOR, FILENAME, DATEEXECUTED, ORDEREXECUTED, MD5SUM, `DESCRIPTION`, COMMENTS, EXECTYPE, CONTEXTS, LABELS, LIQUIBASE, DEPLOYMENT_ID) VALUES ('V2_3 Drop Primary Name Column', 'howarddeiner', 'src/main/resources/db/migration/V2_3__Drop_Primary_Name_Column.xml', NOW(), 5, '8:409c536a5414ff2f6a24eb0ad16ba3f2', 'sql', '', 'EXECUTED', NULL, NULL, '3.8.7', '4641318752')
[INFO] Successfully released change log lock
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 3.175 s
[INFO] Finished at: 2020-03-19T14:08:39-04:00
[INFO] Final Memory: 18M/342M
[INFO] ------------------------------------------------------------------------
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building advanced-liquibase-migrations-sql-only 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- liquibase-maven-plugin:3.8.7:dbDoc (default-cli) @ advanced-liquibase-migrations-sql-only ---
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] 
[INFO] Liquibase Community 3.8.7 by Datical
[INFO] Starting Liquibase at Thu, 19 Mar 2020 14:08:41 EDT (version 3.8.7 #55 built at Mon Feb 24 03:04:51 UTC 2020)
[INFO] Executing on Database: jdbc:mysql://localhost:3306/zipster?useSSL=false
[INFO] Generating Database Documentation
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOGLOCK
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOGLOCK
[INFO] SELECT `LOCKED` FROM DATABASECHANGELOGLOCK WHERE ID=1
[INFO] Successfully acquired change log lock
[INFO] SELECT MD5SUM FROM DATABASECHANGELOG WHERE MD5SUM IS NOT NULL LIMIT 1
[INFO] SELECT COUNT(*) FROM DATABASECHANGELOG
[INFO] Reading from DATABASECHANGELOG
[INFO] SELECT * FROM DATABASECHANGELOG ORDER BY DATEEXECUTED ASC, ORDEREXECUTED ASC
[INFO] Successfully released change log lock
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2.759 s
[INFO] Finished at: 2020-03-19T14:08:43-04:00
[INFO] Final Memory: 17M/358M
[INFO] ------------------------------------------------------------------------
Using defaults:
   show_timings True
  show_snippets True
 logging_format %(levelname)s:%(name)s:%(message)s
          junit False
          color True
   default_tags 
       userdata {}
        summary True
   show_skipped True
        dry_run False
scenario_outline_annotation_schema {name} -- @{row.id} {examples.name}
    log_capture True
 stdout_capture True
  steps_catalog False
 stderr_capture True
          stage None
  logging_level 20
 default_format pretty
    show_source True
Supplied path: "features/step_3_tests.feature"
Primary path is to a file so using its directory
Trying base directory: /home/howarddeiner/IdeaProjects/advanced-liquibase-migrations-sql-only/src/test/python/features
Feature: Version 2 Database # features/step_3_tests.feature:1

  Scenario: Ensure that the V2_1, V2_2, and V2_3 Liquibase migrations happened correctly  # features/step_3_tests.feature:3
    Given the database is at version "V2_3"                                               # features/steps/common_steps.py:6 0.000s
    Then the "zipster" database schema should be                                          # features/steps/common_steps.py:10
      | TABLE_NAME            | COLUMN_NAME       | ORDINAL_POSITION | DATA_TYPE | CHARACTER_MAXIMUM_LENGTH |
      | DATABASECHANGELOG     | ID                | 1                | varchar   | 255                      |
      | DATABASECHANGELOG     | AUTHOR            | 2                | varchar   | 255                      |
      | DATABASECHANGELOG     | FILENAME          | 3                | varchar   | 255                      |
      | DATABASECHANGELOG     | DATEEXECUTED      | 4                | datetime  | NULL                     |
      | DATABASECHANGELOG     | ORDEREXECUTED     | 5                | int       | NULL                     |
      | DATABASECHANGELOG     | EXECTYPE          | 6                | varchar   | 10                       |
      | DATABASECHANGELOG     | MD5SUM            | 7                | varchar   | 35                       |
      | DATABASECHANGELOG     | DESCRIPTION       | 8                | varchar   | 255                      |
      | DATABASECHANGELOG     | COMMENTS          | 9                | varchar   | 255                      |
      | DATABASECHANGELOG     | TAG               | 10               | varchar   | 255                      |
      | DATABASECHANGELOG     | LIQUIBASE         | 11               | varchar   | 20                       |
      | DATABASECHANGELOG     | CONTEXTS          | 12               | varchar   | 255                      |
      | DATABASECHANGELOG     | LABELS            | 13               | varchar   | 255                      |
      | DATABASECHANGELOG     | DEPLOYMENT_ID     | 14               | varchar   | 10                       |
    Then the "zipster" database schema should be                                          # features/steps/common_steps.py:10 0.009s
      | TABLE_NAME            | COLUMN_NAME       | ORDINAL_POSITION | DATA_TYPE | CHARACTER_MAXIMUM_LENGTH |
      | DATABASECHANGELOG     | ID                | 1                | varchar   | 255                      |
      | DATABASECHANGELOG     | AUTHOR            | 2                | varchar   | 255                      |
      | DATABASECHANGELOG     | FILENAME          | 3                | varchar   | 255                      |
      | DATABASECHANGELOG     | DATEEXECUTED      | 4                | datetime  | NULL                     |
      | DATABASECHANGELOG     | ORDEREXECUTED     | 5                | int       | NULL                     |
      | DATABASECHANGELOG     | EXECTYPE          | 6                | varchar   | 10                       |
      | DATABASECHANGELOG     | MD5SUM            | 7                | varchar   | 35                       |
      | DATABASECHANGELOG     | DESCRIPTION       | 8                | varchar   | 255                      |
      | DATABASECHANGELOG     | COMMENTS          | 9                | varchar   | 255                      |
      | DATABASECHANGELOG     | TAG               | 10               | varchar   | 255                      |
      | DATABASECHANGELOG     | LIQUIBASE         | 11               | varchar   | 20                       |
      | DATABASECHANGELOG     | CONTEXTS          | 12               | varchar   | 255                      |
      | DATABASECHANGELOG     | LABELS            | 13               | varchar   | 255                      |
      | DATABASECHANGELOG     | DEPLOYMENT_ID     | 14               | varchar   | 10                       |
      | DATABASECHANGELOGLOCK | ID                | 1                | int       | NULL                     |
      | DATABASECHANGELOGLOCK | LOCKED            | 2                | bit       | NULL                     |
      | DATABASECHANGELOGLOCK | LOCKGRANTED       | 3                | datetime  | NULL                     |
      | DATABASECHANGELOGLOCK | LOCKEDBY          | 4                | varchar   | 255                      |
      | NAME                  | NCONST            | 1                | varchar   | 31                       |
      | NAME                  | FIRST_NAME        | 2                | varchar   | 255                      |
      | NAME                  | LAST_NAME         | 3                | varchar   | 255                      |
      | NAME                  | BIRTH_YEAR        | 4                | int       | NULL                     |
      | NAME                  | DEATH_YEAR        | 5                | int       | NULL                     |
      | NAME_PROFESSION       | NCONST            | 1                | varchar   | 31                       |
      | NAME_PROFESSION       | PROFESSION        | 2                | varchar   | 31                       |
      | NAME_TITLE            | NCONST            | 1                | varchar   | 31                       |
      | NAME_TITLE            | TCONST            | 2                | varchar   | 31                       |
      | TITLE                 | TCONST            | 1                | varchar   | 31                       |
      | TITLE                 | TITLE_TYPE        | 2                | varchar   | 31                       |
      | TITLE                 | PRIMARY_TITLE     | 3                | varchar   | 1023                     |
      | TITLE                 | ORIGINAL_TITLE    | 4                | varchar   | 1023                     |
      | TITLE                 | IS_ADULT          | 5                | bit       | NULL                     |
      | TITLE                 | START_YEAR        | 6                | int       | NULL                     |
      | TITLE                 | END_YEAR          | 7                | int       | NULL                     |
      | TITLE                 | RUNTIME_MINUTES   | 8                | int       | NULL                     |
      | TITLE_AKA             | TCONST            | 1                | varchar   | 31                       |
      | TITLE_AKA             | ORDERING          | 2                | int       | NULL                     |
      | TITLE_AKA             | TITLE             | 3                | varchar   | 1023                     |
      | TITLE_AKA             | REGION            | 4                | varchar   | 15                       |
      | TITLE_AKA             | LANGUAGE          | 5                | varchar   | 63                       |
      | TITLE_AKA             | TYPES             | 6                | varchar   | 63                       |
      | TITLE_AKA             | ATTRIBUTES        | 7                | varchar   | 63                       |
      | TITLE_AKA             | IS_ORIGINAL_TITLE | 8                | bit       | NULL                     |
      | TITLE_DIRECTOR        | TCONST            | 1                | varchar   | 31                       |
      | TITLE_DIRECTOR        | NCONST            | 2                | varchar   | 31                       |
      | TITLE_EPISODE         | TCONST            | 1                | varchar   | 31                       |
      | TITLE_EPISODE         | TCONST_PARENT     | 2                | varchar   | 31                       |
      | TITLE_EPISODE         | SEASON_NUMBER     | 3                | int       | NULL                     |
      | TITLE_EPISODE         | EPISODE_NUMBER    | 4                | int       | NULL                     |
      | TITLE_GENRE           | TCONST            | 1                | varchar   | 31                       |
      | TITLE_GENRE           | GENRE             | 2                | varchar   | 31                       |
      | TITLE_PRINCIPALS      | TCONST            | 1                | varchar   | 31                       |
      | TITLE_PRINCIPALS      | ORDERING          | 2                | int       | NULL                     |
      | TITLE_PRINCIPALS      | NCONST            | 3                | varchar   | 31                       |
      | TITLE_PRINCIPALS      | CATEGORY          | 4                | varchar   | 63                       |
      | TITLE_PRINCIPALS      | JOB               | 5                | varchar   | 255                      |
      | TITLE_PRINCIPALS      | CHARACTER_PLAYED  | 6                | varchar   | 255                      |
      | TITLE_RATING          | TCONST            | 1                | varchar   | 31                       |
      | TITLE_RATING          | AVERAGE_RATING    | 2                | varchar   | 15                       |
      | TITLE_RATING          | NUMBER_OF_VOTES   | 3                | int       | NULL                     |
      | TITLE_WRITER          | TCONST            | 1                | varchar   | 31                       |
      | TITLE_WRITER          | NCONST            | 2                | varchar   | 31                       |
    And the following tables have the following row counts                                # features/steps/common_steps.py:38 0.039s
      | TABLE_NAME       | ROW_COUNT |
      | NAME             | 6234      |
      | NAME_PROFESSION  | 16300     |
      | NAME_TITLE       | 24389     |
      | TITLE            | 3735      |
      | TITLE_AKA        | 2572      |
      | TITLE_DIRECTOR   | 3620      |
      | TITLE_EPISODE    | 2540      |
      | TITLE_GENRE      | 7228      |
      | TITLE_PRINCIPALS | 21255     |
      | TITLE_RATING     | 612       |
      | TITLE_WRITER     | 2449      |
    And the NAME table has the following sample results                                   # features/steps/step_3_tests.py:5 0.021s
      | NCONST    | FIRST_NAME | LAST_NAME | BIRTH_YEAR | DEATH_YEAR |
      | nm0000007 | Humphrey   | Bogart    | 1899       | 1957       |
      | nm0000859 | Lionel     | Barrymore | 1878       | 1954       |
      | nm0000093 | Brad       | Pitt      | 1963       | -1         |
      | nm0000678 | Kathleen   | Turner    | 1954       | -1         |
      | nm0000187 | Madonna    |           | 1958       | -1         |
      | nm0001145 | Divine     |           | 1945       | 1988       |

1 feature passed, 0 failed, 0 skipped
1 scenario passed, 0 failed, 0 skipped
4 steps passed, 0 failed, 0 skipped, 0 undefined
Took 0m0.069s
```

The same basic comments about this script being tedious apply from the last script to this one.

It's really quite simple.  We  need three steps to refactor a database that has a VARCHAR column into a database with two VARCHAR columns and preserve design intent.  We can use SQL based migrations to add the two new columns, and remove the superfluous one after a JDBC based migration the understands how to read a string of first and last names from a column and then split it into a first name column and a last name column. 

###### step_4_destroy_mysql.sh
When run, this script
```bash
#!/usr/bin/env bash

figlet -w 200 -f standard "Destroy MySQL docker-composed Environment"

docker-compose -f docker-compose-mysql-and-mysql-data.yml down

sudo -S <<< "password" rm -rf mysql-data
```
produces

```console
 ____            _                     __  __       ____   ___  _           _            _                                                               _ 
|  _ \  ___  ___| |_ _ __ ___  _   _  |  \/  |_   _/ ___| / _ \| |       __| | ___   ___| | _____ _ __       ___ ___  _ __ ___  _ __   ___  ___  ___  __| |
| | | |/ _ \/ __| __| '__/ _ \| | | | | |\/| | | | \___ \| | | | |      / _` |/ _ \ / __| |/ / _ \ '__|____ / __/ _ \| '_ ` _ \| '_ \ / _ \/ __|/ _ \/ _` |
| |_| |  __/\__ \ |_| | | (_) | |_| | | |  | | |_| |___) | |_| | |___  | (_| | (_) | (__|   <  __/ | |_____| (_| (_) | | | | | | |_) | (_) \__ \  __/ (_| |
|____/ \___||___/\__|_|  \___/ \__, | |_|  |_|\__, |____/ \__\_\_____|  \__,_|\___/ \___|_|\_\___|_|        \___\___/|_| |_| |_| .__/ \___/|___/\___|\__,_|
                               |___/          |___/                                                                            |_|                         
 _____            _                                      _   
| ____|_ ____   _(_)_ __ ___  _ __  _ __ ___   ___ _ __ | |_ 
|  _| | '_ \ \ / / | '__/ _ \| '_ \| '_ ` _ \ / _ \ '_ \| __|
| |___| | | \ V /| | | | (_) | | | | | | | | |  __/ | | | |_ 
|_____|_| |_|\_/ |_|_|  \___/|_| |_|_| |_| |_|\___|_| |_|\__|
                                                             
Stopping mysql ... done
Removing mysql ... done
Removing network advanced-liquibase-migrations-sql-only_default
```

Really not much to say.  We undo step 1, and delete stuff that we created in the process to support the Docker'ized MySQL instance.
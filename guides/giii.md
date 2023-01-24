````bash
  #Install Postgres  version >= 12.6
  psql -V  #confirm version installed
````

````bash
    # Find newest guide on postgres official site
    sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/F-37-x86_64/pgdg-fedora-repo-latest.noarch.rpm
    sudo dnf install -y postgresql15-server
    sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
    sudo systemctl enable postgresql-15
    sudo systemctl start postgresql-15
````

````blockquote

    Now cd /var/lib/pgsql/15/data/
    edit pg_hba.conf
    from  
    local   all             all                                     peer
    host    all             all             0.0.0.0/0                   scram-sha-256
    To
    local   all             all                                     trust
    host    all             all             0.0.0.0/0                   trust


    Edit postgresql.conf
    Find and uncomment line #listen_addresses = 'localhost' and edit to
       listen_addresses = '*'
       Uncomment also:
       #port = 5432
       #password_encryption = scram-sha-256     # scram-sha-256 or md5

    Exit from root and restart the service:
    sudo systemctl restart postgresql-15.service
   
    Login to postgres
    psql -U postgres
    run command:
    alter user postgres with password 'YOURPASSWORD';
    Exit from postgres session.
   



    Now cd /var/lib/pgsql/15/data/
    edit pg_hba.comf
    from
    local   all             all                                     trust
    host    all             all             0.0.0.0/0                   trust
    To
    local   all             all                                     scram-sha-256
    host    all             all             0.0.0.0/0                   scram-sha-256
   
    Exit from root and restart the service:
    sudo systemctl restart postgresql-15.service
   
    login to postgres using 'YOURPASSWORD'
````

## Install Wildfly

````
Install Wildfly  version >= 18.0 Final
````

## Configure Datasource & Setup Driver

````xml
 <datasource jndi-name="java:jboss/datasources/PostgresXe" pool-name="PostgresXe" enabled="true" use-java-context="true">
                    <connection-url>jdbc:postgresql://localhost:5432/fm</connection-url>
                    <driver>postgresql</driver>
                    <security>
                        <user-name>postgres</user-name>
                        <password>postgres</password>
                    </security>
                </datasource>
````

#### NB: Replace above with your database name and connection credentials. Use [fm] as database name for smooth run

## Run or deploy Xe

````bash
git checkout xe-postgres
mvn clean compile wildfly:deploy
````

## Confirm Creation of Tables

````bash
 psql -U your_postgres_user
 #Enter Your password
 \l #show databases
 \c your_db #switch to db
 \dt #show tables
 \q #Quit
````

Create this tables

```SQL
create domain clob as text;

create table USERS_SCHEMES
(
    USERS_ID          bigint not null
        constraint FKI664IMXO4SPEB8J43W9RUKF1S
            references USERS,
    ALLOWEDSCHEMES_ID bigint not null
        constraint FKMQV6UDXD3PCOR5T8U6X8DBNX2
            references SCHEMES
);

create table USERS_MEMBER_CLASSES
(
    USERS_ID
        bigint
        not
            null
        constraint
            FK9TGKRTT6A6RH2FHRHBQU0EX0V
            references
                USERS,
    MEMBERCLASSES_ID
        bigint
        not
            null
        constraint
            FKM6XCAGUW6K14CQ9BHUG1DNOPV
            references
                MEMBER_CLASSES
);

create table USERS_SPONSORS
(
    USERS_ID
        bigint
        not
            null
        constraint
            FKT3JSM9FJL7S6UWOMIO41YD0BV
            references
                USERS,
    ALLOWEDSPONSORS_ID
        bigint
        not
            null
        constraint
            FKK7GMGB78K7MSQ912GKO8UIBVU
            references
                SPONSORS
);

```

## Import functions and procedures and Views

````bash
# locate routines_64.sql and views_349.sql in ../resources/pg_scripts/ in project folder
 psql -U your_postgres_user
 # Enter Your password
 \l # show databases
 # You may require to run this next two steps severally to ensure all views and routines are created
 \c your_db # switch to db
 \i path_to_routines_64.sql # import routines, repeat until no errors
 \i path_to_views_349.sql # import views,  repeat till no errors
 \df #confirm routines
 \dv #confirm views
 \q # Quit
````



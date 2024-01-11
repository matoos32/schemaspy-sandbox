# SchemaSpy Sandbox

This is a collection of cookbook notes taken while prototyping with
[SchemaSpy](https://github.com/schemaspy/schemaspy).

## Table of Contents

- [PostgreSQL demo database with remote client](#postgresql-demo-database-with-remote-client)
  - [Assumptions](#assumptions)
  - [Client setup](#client-setup)
    - [Ensure a Java JRE is installed](#ensure-a-java-jre-is-installed)
    - [Initialize local directories](#initialize-local-directories)
    - [Get the PostgreSQL driver](#get-the-postgresql-driver)
    - [Get the SchemaSpy jar](#get-the-schemaspy-jar)
    - [Create a SchemaSpy config file](#create-a-schemaspy-config-file)
    - [Start the local SSH tunnel](#start-the-local-ssh-tunnel)
  - [Run SchemaSpy](#run-schemaspy)
  - [View the output](#view-the-output)

## PostgreSQL demo database with remote client

The strategy is to use a desired SchemaSpy JAR file and PostgreSQL (PG) JDBC driver to generate
output for the *Pagila* sample database. SchemaSpy will connect via an
[SSH tunnel](https://www.postgresql.org/docs/current/ssh-tunnels.html) to the PG host.

### Assumptions

- The PG cluster is already setup and running.

- A user is created in the PG host operating system and who is able to connect via SSH.

- The Pagila demo database is installed in the PG cluster. If not, you can get it from *postgresql.org* [here](https://www.postgresql.org/ftp/projects/pgFoundry/dbsamples/pagila/pagila/) and use [psql](https://www.postgresql.org/docs/current/app-psql.html) to install it.
- A *user* is created with permission to connect to the Pagila database and with read permissions such as *SELECT* on objects in the `public` schema (the permissions needed by SchemaSpy).

### Client setup

#### Ensure a Java JRE is installed

*Example:*

```sh
sudo apt-get install openjdk-21-jre-headless
```

#### Initialize local directories

```sh
mkdir -p ~/databases/drivers
mkdir -p ~/databases/schemaspy/bin
mkdir ~/databases/schemaspy/cfg
mkdir ~/databases/schemaspy/output
```

#### Get the PostgreSQL driver

Find the desired driver at <https://jdbc.postgresql.org/download/> and download it

*Example*:

```sh
wget https://jdbc.postgresql.org/download/postgresql-42.7.1.jar -P ~/databases/drivers/
```

#### Get the SchemaSpy jar

Find the desired SchemaSpy jar at <https://github.com/schemaspy/schemaspy/releases> and download it

*Example*:

```sh
wget https://github.com/schemaspy/schemaspy/releases/download/v6.2.4/schemaspy-6.2.4.jar -P ~/databases/schemaspy/bin/
```

#### Create a SchemaSpy config file

```sh
touch ~/databases/schemaspy/cfg/schemaspy.properties
```

Using the
[Getting Started config file example](https://schemaspy.readthedocs.io/en/latest/started.html) and
[SchemaSpy command-line args](https://schemaspy.readthedocs.io/en/latest/configuration/commandline.html)
for inspiration, set its contents to:

```text
schemaspy.t=pgsql11

# optional path to alternative jdbc drivers.
schemaspy.dp=/home/<os user>/databases/drivers

# database properties: host, port number, name user, password
schemaspy.host=localhost
schemaspy.port=5432
schemaspy.db=pagila
schemaspy.u=<db user>
schemaspy.p=<db password>

# output dir to save generated files
schemaspy.o=/home/<os user>/databases/schemaspy/output

# db schema for which to generate diagrams
schemaspy.s=public

schemaspy.debug=false
```

You'll need to replace the `<os user>`, `<db user>`, and `<db password>` fields.

For the `<os user>`, setting the explicit client host OS user name as `$USER` caused SchemaSpy to
fail attempting to create the output folder.

Maybe the `<db user>` and `<db password>` fields can be specified at the command line instead?

#### Start the local SSH tunnel

```sh
ssh -L 5432:localhost:5432 <db host username>@<db host>
```

The above will forward client localhost traffic to the database host, and within the
database host the traffic will be forwarded to a localhost connection to the DB.

### Run SchemaSpy

```sh
java -jar ~/databases/schemaspy/bin/schemaspy-6.2.4.jar -configFile ~/databases/schemaspy/cfg/schemaspy.properties
```

The output should be in `~/databases/schemaspy/output`

### View the output

SchemaSpy generates a report and diagrams as a system of HTML pages.

You can copy them for viewing locally or if acceptable view them with an HTTP server.

For example Python has a built-in server:

```sh
cd ~/databases/schemaspy/output

python3 -m http.server
```

With the above navigate your browser to `http://<client host>:8000`.

---
order: 4
icon: cloud
title: Hosting
subtitle: Run a Digital ID server and host your data yourself
---

<div class="alert alert-warning">
    <h4>Warning and Disclaimer</h4>
    <p>We recommend <strong>not</strong> to run the current Digital ID Server in production. See the <a href="{% link library.md %}#roadmap">roadmap</a> for more information.</p>
</div>

## Requirements

### Java Runtime

The Digital ID Server requires the [Java SE Runtime Environment 8](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html).
You can check your current version (and whether you already have Java) with `java -version` on the command line of your operating system.

<div class="alert alert-info">
    <p>Please note that the version numbering of Java is decimal. All you need is a version of 1.8 or higher.</p>
</div>

### Database System

Make sure that you have either [MySQL](https://www.mysql.com) or [MariaDB](https://mariadb.com) installed and that you know the password to access your database.
[PostgreSQL](https://www.postgresql.org) is not supported in the current version because of locking issues.
If you have the necessary expertise, we would be thrilled if you could help us [fixing it]({% link community.md %}#contributions).

### Domain Name

In order to set up a host on your server, you need to have a registered domain and configure a subdomain with the name 'id' (e.g. id.example.com) that references your server (either as an 'A' or 'CNAME' [record](https://en.wikipedia.org/wiki/List_of_DNS_record_types)).

<div class="alert alert-info">
    <p>Please note that the address of a host in Digital ID does not contain this subdomain (i.e. it is only example.com). The subdomain is only used internally for resolving host addresses but never presented to the user.</p>
</div>

### Port Number

The server has to be reachable on the [port number](https://en.wikipedia.org/wiki/Port_(computer_networking)) {{ site.data.protocol.port }}.
Make sure that no firewall or router blocks the access and add corresponding exceptions or [forwarding rules](https://en.wikipedia.org/wiki/Port_forwarding) otherwise.

### Apache Maven

Since this is currently only a test version, you need [Apache Maven](https://maven.apache.org) to build the server from the source code.
A proper distribution package is planned for the future (and is also something you could [help us]({% link community.md %}#contributions) with).

## Installation

### Download

Clone the [utility](https://github.com/synacts/digitalid-utility), [database](https://github.com/synacts/digitalid-database) and [core](https://github.com/synacts/digitalid-core) projects from [GitHub](https://github.com/):

```bash
git clone https://github.com/synacts/digitalid-utility.git
git clone https://github.com/synacts/digitalid-database.git
git clone https://github.com/synacts/digitalid-core.git
```

### Checkout

Checkout the development branch of each repository as follows:
```bash
git -C digitalid-utility checkout development
git -C digitalid-database checkout development
git -C digitalid-core checkout development
```

### Compilation

Build the source code as follows:

```bash
mvn -f digitalid-utility/pom.xml clean install
mvn -f digitalid-database/pom.xml clean install
mvn -f digitalid-core/pom.xml clean install
```

## Execution

### Startup

The server can be started from the [command line](https://en.wikipedia.org/wiki/Command-line_interface) as follows:

```bash
mvn -f digitalid-core/server/pom.xml -q exec:java
```

If you start the server for the first time, it prompts you to configure the database.

### Usage

Once the server has started up, you enter an infinite loop that looks like this:

```
You have the following options:
–  0: Exit the server.
–  1: Show the version.
–  2: Show the hosts.
–  3: Create a host.
–  4: Export a host.
–  5: Import a host.
–  6: Show the services.
–  7: Reload the services.
–  8: Activate a service.
–  9: Deactivate a service.
– 10: Change a provider.
– 11: Generate tokens.
– 12: Show members.
– 13: Add members.
– 14: Remove members.
– 15: Open a host.
– 16: Close a host.

Execute the option: _
```

### Hosts

You can now choose the option 3 to create a host.
Enter your domain name without the subdomain 'id' (i.e. only example.com).
A host manages all identities and their associated data at the specified domain.
You need a separate host for each domain, which can all be run by the same server on a single machine (similar to modern webservers).

<div class="alert alert-info">
    <p>Please note that it can take several minutes to generate the cryptographic keys during the creation of a host.</p>
</div>

### Services

At the moment, the functionality of the Digital ID Server cannot be extended with [services]({% link protocol.html %}#services).  
(This only worked in an [older version]({% link library.md %}#history) of the library.
See the [roadmap]({% link library.md %}#roadmap) for when this will be supported again.)

### Directory

All files created by the Digital ID Server are stored by default in an invisible folder in the user's home directory with the name `~/.digitalid`.
This directory has the following subfolders:
- **hosts**: Contains the public and private keys of all the hosts that run on this server.  
(The keys of the host created above should now be listed in this directory.)
- **clients**: Contains the client secrets of all the clients on this machine.  
(Secrets that begin with an underscore belong to the client of the corresponding host.)
- **services**: Contains the implementations of additional services as [JAR files](https://en.wikipedia.org/wiki/JAR_(file_format)).
- **data**: Contains the database configuration of the server.  
(It also contains the database files if clients are run with [SQLite](https://sqlite.org)).
- **logs**: Contains the log files of the server (and client).  
(A new file with the current date as its name is created each day.)

<div class="alert alert-info">
    <p>Please note that the library does not adhere to the above list of folders at the moment.</p>
</div>

### Certification

Before clients can connect to your host, you need to have its public key certified.
Unfortunately, this process is not supported yet, which is one of the main reasons why you should only use the server for testing at the moment.
Have a look at the [roadmap]({% link library.md %}#roadmap) for more information on this.

### Deinstallation

All you have to do to deinstall the Digital ID Server and remove its data is to drop the database, delete the downloaded source directories and remove the hidden folders `~/.m2/repository/net/digitalid` and `~/.digitalid`.

### Troubleshooting

If you run into problems, have a look at the log files in `~/.digitalid/logs`.
You can adjust the logging threshold in `~/.digitalid/configs/logging.conf` with the logging levels being `Verbose`, `Debugging`, `Information`, `Warning`, `Error`, `Fatal` and `Off`.

## Support

Please file any issues you encounter directly in the corresponding GitHub project ([utility](https://github.com/synacts/digitalid-utility/issues), [database](https://github.com/synacts/digitalid-database/issues) or [core](https://github.com/synacts/digitalid-core/issues)).
If you are uncertain to which project an issue belongs, use the [last one](https://github.com/synacts/digitalid-core/issues) or [contact us](mailto:support@digitalid.net) directly.
When you do so, please attach the current [log file](#troubleshooting).
(Since the log file might contain sensitive information, it might be a good idea to first have a look at it yourself.)

## License

The code of the Digital ID Server is available under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0), which is a [permissive software license](https://en.wikipedia.org/wiki/Permissive_software_licence) that lets you do anything you want with the code as long as you provide proper attribution and don’t hold us liable.

{% comment %}

## Scaling

### Load Balancing

### Database Cluster

### Failover Mechanism

{% endcomment %}

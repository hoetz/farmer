---
title: "SQL Azure"
date: 2020-02-05T08:53:46+01:00
weight: 19
chapter: false
---

#### Overview
The SQL Azure module contains two builders - `sqlServer`, used to create SQL Azure servers, and `sqlDb`, used to create individual databases. It supports features such as encryption, firewalls and automatic pool creation. Every SQL Azure server you create will automatically create a SecureString parameter for the admin account password.

* SQL Azure server (`Microsoft.Sql/servers`)

#### SQL Server Builder Keywords
| Keyword | Purpose |
|-|-|
| server_name | Sets the name of the SQL server. |
| add_firewall_rule | Adds a custom firewall rule given a name, start and end IP address range. |
| enable_azure_firewall | Adds a firewall rule that enables access to other Azure services. |
| admin_username | Sets the admin username of the server. |
| elastic_pool_name | Sets the name of the elastic pool, if required. If not set, Farmer will generate a name for you. |
| elastic_pool_sku | Sets the sku of the elastic pool, if required. If not set, Farmer will default to Basic 50. |
| elastic_pool_database_min_max | Sets the optional minimum and maximum DTUs for the elastic pool for each database. |
| elastic_pool_capacity | Sets the optional disk size in MB for the elastic pool for each database. |

#### SQL Server Configuration Members
| Member | Purpose |
|-|-|
| ConnectionString | Gets a literal .NET connection string using the administrator username / password, given a database or database name. The password will be evaluated based on the contents of the password parameter supplied to the template at deploy time. |

#### SQL Database Builder Keywords

| Keyword | Purpose |
|-|-|
| name | Sets the name of the database. |
| sku | Sets the sku of the database. If not set, the database is assumed to be part of an elastic pool which will be automatically created. |
| collation | Sets the collation of the database. |
| use_encryption | Enables transparent data encryption of the database. |

#### Example
```fsharp
open Farmer
open Farmer.Builders
open Sql

let myDatabases = sqlServer {
    name "my_server"
    admin_username "admin_username"
    enable_azure_firewall

    elastic_pool_name "mypool"
    elastic_pool_sku PoolSku.Basic100

    add_databases [
        sqlDb { name "poolDb1" }
        sqlDb { name "poolDb2" }
        sqlDb { name "standaloneDb1"; sku DbSku.Basic }
    ]
}

let template = arm {
    location Location.NorthEurope
    add_resource myDatabases
}

template
|> Writer.quickWrite "sql-example"

template
|> Deploy.execute "my-resource-group" [ "password-for-my_server", "*****" ]
```
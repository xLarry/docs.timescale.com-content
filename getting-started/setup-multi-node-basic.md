# Set up multi-node TimescaleDB [](basic-multi-node-setup)

A multi-node TimescaleDB implementation consists of: 
- One access node to handle ingest, data routing and act as an entry 
point for user access; 
- One or more data nodes to store and organize distributed data.

All nodes begin as standalone TimescaleDB instances, i.e., hosts with
a running PostgreSQL server and a loaded TimescaleDB extension. This
is assumed for "access node" and "data node" in the instructions. More
detail on the architecture can be found in the
[Architecture][architecture] section.

TimescaleDB multi-node can be created as part of a self-managed deployment
or (coming soon) as a managed cloud deployment.  In order to set up a
self-managed cluster, including how to configure the nodes for secure
communication and creating users/roles across servers, please follow
[these instructions][advanced setup] before proceeding.

In the case of Timescale Cloud and Forge, the created services already contain
PostgreSQL with TimescaleDB loaded and the created user `tsdbadmin` as superuser.
In this case, all you will need to do is decide which service should be the access
node, and follow the instructions in the next section.  More information will be
forthcoming as TimescaleDB multi-node is made available on these cloud platforms.

## Initialize data nodes from the access node [](init_data_nodes_on_access_node)

Once logged in on the access node, it is necessary to add data nodes
to the local database before creating distributed hypertables. This
will make the data node available for use by distributed hypertables
and the access node will also connect to the data node and initialize
it.

While connected to the access node as a superuser (e.g., via `psql`),
use the command:

```sql
SELECT add_data_node('example_node_name', host => 'example_host_address');
```

`example_node_name` should be a unique name for the
node. `example_host_address` is the host name or IP address of the
data node. You can specify a password to authenticate with using the
optional `password` parameter. But this is only necessary if password
authentication is used to connect to data nodes and the password is
not provided through other means (e.g., a local password file). See
the [`add_data_node`][add_data_node] API reference documentation for
detailed information about this command.

You can now create distributed hypertables using
`create_distributed_hypertable`, but note that, in order to create and
use distributed hypertables as a non-superuser, the user role needs to
exist on all data nodes and have the correct privileges to use the
data nodes on the access node. Please refer to the next section for
instruction on how to create roles on all data nodes and grant data
node privileges.

It should now be possible to create distributed hypertables, e.g.,
running as the default `postgres` superuser. If you wish to create and
use distributed hypertables as a non-superuser, the user role needs to
be distributed as well (i.e., exist on all data nodes) and requires
privileges to use the data nodes on the access node. Please refer to
the next section for instructions on how to create distributed roles
and grant data node privileges.

## (Optional) Add roles to all data nodes

When you add a role on the access node it will not be automatically
created on the data nodes. Therefore, the role must also be created on
the data nodes before it can be used to create and query distributed
hypertables:

```sql
CREATE ROLE testrole;
CALL distributed_exec($$ CREATE ROLE testrole WITH LOGIN $$);
```

Note that, depending on how the access node authenticates with the data
nodes, the new role might need to be configured with, e.g., a
password. Please refer to the following sections for more specific
instructions depending on the authentication mechanism you are using:
- Adding roles using [trust authentication][trust_role_setup]
- Adding roles using [password authentication][password_role_setup]
- Adding roles using [certificate authentication][certificate_role_setup]

Finally, grant data node privileges to the user role:


```sql
GRANT USAGE ON FOREIGN SERVER <data node name>, <data node name>, ... TO testrole;
```

>:TIP: It is possible to grant data node usage to `PUBLIC`, in which
>case all user roles (including future ones) will be able to use the
>specified data nodes.

---
## Next steps
To start using the database, see the page on [distributed hypertables][].

To further configure the system (set up secure node-to-node communication, add 
additional users/roles) see [advanced setup][].

All functions for modifying the node network are described in the API
docs:
- [add_data_node][]
- [attach_data_node][]
- [delete_data_node][]
- [detach_data_node][]
- [distributed_exec][]

[architecture]: /introduction/architecture#single-node-vs-clustering
[install]: /getting-started/installation
[setup]: /getting-started/setup
[advanced setup]: /getting-started/setup-multi-node-basic/setup-multi-node-auth
[trust_role_setup]: /getting-started/setup-multi-node-basic/setup-multi-node-auth#multi-node-auth-trust-roles
[password_role_setup]: /getting-started/setup-multi-node-basic/setup-multi-node-auth#multi-node-auth-password-roles
[certificate_role_setup]: /getting-started/setup-multi-node-basic/setup-multi-node-auth#multi-node-auth-certificate-roles
[postgresql-hba]: https://www.postgresql.org/docs/current/auth-pg-hba-conf.html
[max_prepared_transactions]: https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-MAX-PREPARED-TRANSACTIONS
[distributed hypertables]: /using-timescaledb/distributed-hypertables
[add_data_node]: /api#add_data_node
[attach_data_node]: /api#attach_data_node
[delete_data_node]: /api#delete_data_node
[detach_data_node]: /api#detach_data_node
[distributed_exec]: /api#distributed_exec

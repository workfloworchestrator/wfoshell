# wfoshell

The wfoshell (WorkFlow Orchestrator Shell) is an interactive shell to navigate
through subscriptions product blocks and resource types, and update
subscriptions and resource types directly in the database. The use of the GNU
Readline interface allows for command completion and history, as wel as command
line editing and search.

For various reasons, it sometimes happens that incorrect information ends up in
the WFO database. Those who know the database model can of course adjust this
information directly in the database with self-made SQL queries. For those who
prefer an easy way to navigate through the subscriptions, product blocks and
resource types to adjust incorrect information, can use the wfoshell.

## Getting started

The wfoshell is tested with Python version 3.12.

Install the required Python modules:

```shell
pip install -r requirements/base.txt
```

And start the shell:

```shell
python main.py
```

## Warning

The shell operates directly on the database, changes made are instantly
commited to the database. While using the shell, try to avoid other write
access to the database, or at least limit write access to the information you
are touching. Also note that none of the information that is updated in the
database is checked syntactically or in any other way, except for the insync,
start_date and end_date subscription fields, these fields will not allow
syntactically incorrect values.  Updating information in the database with
unsupported values may brake things. Use this shell at your own risk.

Only scalar resource types are supported. All non-scalar resource types are
shown as `<unset or non-scalar>` while they can have a value in the database.
Optional yet unset resource types can be assigned a value with the
`resource_type  update` command, but do not try to update non-scalar resource
types using the wfoshell.

## Usage

### Command overview

The following commands are supported. Each command has its own help on their
subcommands.

```text
(wfo) help --verbose

Documented commands (use 'help -v' for verbose/'help <topic>' for details):
======================================================================================================
exit                  Exit the application.
help                  List available commands or provide detailed help for a specific command
history               View, run, edit, save, or clear previously entered commands
product_block         List and select product blocks, show details, or follow depends on and in use by
                      product blocks.
quit                  Exit this application
resource_type         List, select and update resource types, and show details.
set                   Set a settable parameter or show current settings of parameters
state                 Show state summary or details.
subscription          List, search or select subscriptions, update fields, and show details.
```

The `subscription`, `product_block` and `resource_type` commands are used
to navigate through the database and update information. All three commands
have `list`, `select` and `details` subcommands, and the `subscription`
and `resource_type` commands have an `update` subcommand. These subcommands
are used to list the specific type of information, select an item to work with,
show more detailed information, and update information in the database. In
addition, the `subscription` command has a case insensitive `search`
subcommand to quickly find a subscription, and the `product_block` command
has `depends_on` and `in_use_by` subcommands to navigate through product
blocks and therewith through subscriptions.

### Examples

#### Select subscription to update description

This example shows how to list and select a subscription, show subscription
details, and update the subscription description. When the description is
generated by a workflow and uses other information like resource types, then
make sure to match the description witch the associated information.

```text
(wfo) subscription list
0  core link 100G paris-1 ethernet-1/10 <-> ethernet-1/13 amsterdam-1  581a2251-d9cf-4a32-a5d7-1549eb3280e2
1  node amsterdam-1 (active)                                           bf5dacfa-0e89-4f5e-8b2f-d6e9327b21dc
2  node paris-1 (active)                                               05b7198c-9d6e-4a97-9a3e-001e337110ee
3  node rome-1 (active)                                                b19ab933-48d6-4851-9687-41b72c0ab8b6
4  port 10G paris-1 ethernet-1/5 data center connection                fa6b26ed-fee1-48f6-a6ea-6baf39589be1
(wfo) subscription select 3
subscription  node rome-1 (staged)  b19ab933-48d6-4851-9687-41b72c0ab8b6
(wfo) subscription details
description       node rome-1 (staged)
subscription_id   b19ab933-48d6-4851-9687-41b72c0ab8b6
status            active
product_id        e537723f-6399-4ed3-b7ca-c54db4ff2c2a
customer_id       bcaf232c-2f4c-476c-8c9b-c14542e3607d
insync            False
start_date        2024-12-03 08:43:40.476615+00:00
end_date
note
product block(s)  0  name            Node
                     resource types  0  ims_id            3
                                     1  ipv4_ipam_id      9
                                     2  ipv6_ipam_id      10
                                     3  node_description  Rome international exchange
                                     4  node_name         rome-1
                                     5  node_status       staged
                                     6  nrm_id            3555
                                     7  role_id           2
                                     8  site_id           5
                                     9  type_id           7
(wfo) subscription update insync True
(wfo)
```

#### Start with a port to find the node it is on

Search subscription by description and use the product block command to find
the associated node.

```text
(wfo) subscription search "port 10g"
0  port 10G paris-1 ethernet-1/5 data center connection  fa6b26ed-fee1-48f6-a6ea-6baf39589be1
(wfo) subscription select 0
subscription  port 10G paris-1 ethernet-1/5 data center connection  fa6b26ed-fee1-48f6-a6ea-6baf39589be1
(wfo) subscription details --product_blocks_only
product block(s)  0  name            Port
                     resource types  0  auto_negotiation  False
                                     1  enabled           True
                                     2  ims_id            12
                                     3  lldp              False
                                     4  node              <unset or non-scalar>
                                     5  nrm_id            33521
                                     6  port_description  data center connection
                                     7  port_mode         tagged
                                     8  port_name         ethernet-1/5
                                     9  port_type         10gbase-x-xfp
(wfo) product_block select 0
subscription   port 10G paris-1 ethernet-1/5 data center connection  fa6b26ed-fee1-48f6-a6ea-6baf39589be1
product block  Port                                                  2918af3f-7dbb-4408-9fb1-ad71920487bc
(wfo) product_block details --depends_on_only
depends_on  0  name            Node
               resource types  0  ims_id            2
                               1  ipv4_ipam_id      5
                               2  ipv6_ipam_id      6
                               3  node_description  Paris 1
                               4  node_name         paris-1
                               5  node_status       staged
                               6  nrm_id            47469
                               7  role_id           2
                               8  site_id           2
                               9  type_id           5
(wfo) product_block depends_on 0
subscription   node paris-1 (active)  05b7198c-9d6e-4a97-9a3e-001e337110ee
product block  Node                   d097cf20-9c4c-4ba8-9008-701841933a45
(wfo)
```

#### Select a resource type to update its value

Select a resource types on the currently selected product block to update its
value.  New values are not syntactically checked. Any relation that this
resource type has with other information in the WFO database or any related
external administration, should eventually match the new value to avoid
unexpected results using the WFO.

```text
(wfo) state summary
subscription   node paris-1 (active)  05b7198c-9d6e-4a97-9a3e-001e337110ee
product block  Node                   d097cf20-9c4c-4ba8-9008-701841933a45
(wfo) resource_type list
0  ims_id            2
1  ipv4_ipam_id      5
2  ipv6_ipam_id      6
3  node_description  Paris 1
4  node_name         paris-1
5  node_status       staged
6  nrm_id            47469
7  role_id           2
8  site_id           2
9  type_id           5
(wfo)
(wfo) resource_type select 5
subscription   node paris-1 (active)  05b7198c-9d6e-4a97-9a3e-001e337110ee
product block  Node                   d097cf20-9c4c-4ba8-9008-701841933a45
resource_type  node_status            75cc0264-f3f1-49ba-94ad-4bc0b96a5834
(wfo) resource_type update active
(wfo) resource_type details
resource_type                   node_status
value                           active
subscription_instance_value_id  75cc0264-f3f1-49ba-94ad-4bc0b96a5834
subscription_instance_id        d097cf20-9c4c-4ba8-9008-701841933a45
resource_type_id                87a1523a-55d7-4431-804e-c70143330083
(wfo)
```

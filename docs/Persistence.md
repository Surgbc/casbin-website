---
id: persistence
title: Persistence
---

## Model persistence

Unlike the policy, the model can be loaded only, it cannot be saved. Because we think the model is not a dynamic component and should not be modified at run-time, so we don't implement an API to save the model into a storage.

However, the good news is, we provide several ways to load a model statically or dynamically:

1. Load from a model file (.CONF). This is the most common way to use Casbin:

    ```go
    e := casbin.NewEnforcer("examples/basic_model.conf", "examples/basic_policy.csv")
    ```

2. Load from code. Please see: https://github.com/casbin/casbin/wiki/Load-model-from-string

3. Load from string. Please see: https://github.com/casbin/casbin/wiki/Load-model-from-string

### Load model from string

The model can be initialized from code instead of using ``.CONF`` file. Here's an example for the RBAC model:

```go
// Initialize the model from Go code.
m := casbin.NewModel()
m.AddDef("r", "r", "sub, obj, act")
m.AddDef("p", "p", "sub, obj, act")
m.AddDef("g", "g", "_, _")
m.AddDef("e", "e", "some(where (p.eft == allow))")
m.AddDef("m", "m", "g(r.sub, p.sub) && r.obj == p.obj && r.act == p.act")

// Load the policy rules from the .CSV file adapter.
// Replace it with your adapter to avoid files.
a := persist.NewFileAdapter("examples/rbac_policy.csv")

// Create the enforcer.
e := casbin.NewEnforcer(m, a)
```

Or you can just get the model from one single string:

```go
// Initialize the model from a string.
text :=
`
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[role_definition]
g = _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub) && r.obj == p.obj && r.act == p.act
`
m := NewModel(text)

// Load the policy rules from the .CSV file adapter.
// Replace it with your adapter to avoid files.
a := persist.NewFileAdapter("examples/rbac_policy.csv")

// Create the enforcer.
e := casbin.NewEnforcer(m, a)
```

These two ways are equivalent with the common usage:

[examples/rbac_model.conf](https://github.com/casbin/casbin/blob/master/examples/rbac_model.conf):

```ini
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[role_definition]
g = _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub) && r.obj == p.obj && r.act == p.act
```

```go
e := casbin.NewEnforcer("examples/rbac_model.conf", "examples/rbac_policy.csv")
```

## Policy persistence

In Casbin, the policy storage is implemented as an adapter (aka middleware for Casbin). A Casbin user can use an adapter to load policy rules from a storage (aka ``LoadPolicy()``), or save policy rules to it (aka ``SavePolicy()``). To keep light-weight, we don't put adapter code in the main library.

All supported adapters can be found in: https://github.com/casbin/casbin/wiki/Supported-adapters

Here are some things you need to know:

1. If ``casbin.NewEnforcer()`` is called with an explicit or implicit adapter, the policy will be loaded automatically.
2. You can call ``e.LoadPolicy()`` to reload the policy rules from the storage.
3. If the adapter does not support the ``Auto-Save`` feature, The policy rules cannot be automatically saved back to the storage when you add or remove policies. You can to call ``SavePolicy()`` manually to save all policy rules.

### Examples

Here we provide several examples:

#### File adapter (built-in)

Below shows how to initialize an enforcer from the built-in file adapter:

```go
import "github.com/casbin/casbin"

e := casbin.NewEnforcer("examples/basic_model.conf", "examples/basic_policy.csv")
```

This is the same with:

```go
import (
    "github.com/casbin/casbin"
    "github.com/casbin/casbin/file-adapter"
)

a := fileadapter.NewAdapter("examples/basic_policy.csv")
e := casbin.NewEnforcer("examples/basic_model.conf", a)
```

#### [MySQL adapter](https://github.com/casbin/mysql-adapter)

Below shows how to initialize an enforcer from MySQL database. it connects to a MySQL DB on 127.0.0.1:3306 with root and blank password.

```go
import (
    "github.com/casbin/casbin"
    "github.com/casbin/mysql-adapter"
)

a := mysqladapter.NewAdapter("mysql", "root:@tcp(127.0.0.1:3306)/")
e := casbin.NewEnforcer("examples/basic_model.conf", a)
```

### Use your own storage adapter

You can use your own adapter like below:

```go
import (
    "github.com/casbin/casbin"
    "github.com/your-username/your-repo"
)

a := yourpackage.NewAdapter(params)
e := casbin.NewEnforcer("examples/basic_model.conf", a)
```

### Load/Save at run-time

You may also want to reload the model, reload the policy or save the policy after initialization:

```go
// Reload the model from the model CONF file.
e.LoadModel()

// Reload the policy from file/database.
e.LoadPolicy()

// Save the current policy (usually after changed with Casbin API) back to file/database.
e.SavePolicy()
```

### Supported adapters

In Casbin, the policy storage is implemented as an adapter (aka middleware for Casbin). To keep light-weight, we don't put adapter code in the main library. A complete list of Casbin adapters is provided as below.

**Note**: Any 3rd-party contribution on a new adapter is welcomed, please inform us and we will put it in this list:)

Adapter | Type | Author | Description
----|------|----|----
[File Adapter (built-in)](https://github.com/casbin/casbin/wiki/Policy-persistence#file-adapter) | File | Casbin | Persistence for [.CSV (Comma-Separated Values)](https://en.wikipedia.org/wiki/Comma-separated_values) files
[Xorm Adapter](https://github.com/casbin/xorm-adapter) | ORM | Casbin | MySQL, PostgreSQL, TiDB, SQLite, SQL Server, Oracle are supported by [Xorm](https://github.com/go-xorm/xorm/)
[Gorm Adapter](https://github.com/casbin/gorm-adapter) | ORM | Casbin | MySQL, PostgreSQL, Sqlite3, SQL Server are supported by [Gorm](https://github.com/jinzhu/gorm/)
[Beego ORM Adapter](https://github.com/casbin/beego-orm-adapter) | ORM | Casbin | MySQL, PostgreSQL, Sqlite3 are supported by [Beego ORM](https://beego.me/docs/mvc/model/overview.md)
[MongoDB Adapter](https://github.com/casbin/mongodb-adapter) | NoSQL | Casbin | Persistence for [MongoDB](https://www.mongodb.com)
[Cassandra Adapter](https://github.com/casbin/cassandra-adapter) | NoSQL | Casbin | Persistence for [Apache Cassandra DB](http://cassandra.apache.org)
[Consul Adapter](https://github.com/ankitm123/consul-adapter) | KV store | [@ankitm123](https://github.com/ankitm123) | Persistence for [HashiCorp Consul](https://www.consul.io/)
[Redis Adapter](https://github.com/casbin/redis-adapter) | KV store | Casbin | Persistence for [Redis](https://redis.io/)
[Protobuf Adapter](https://github.com/casbin/protobuf-adapter) | Stream | Casbin | Persistence for [Google Protocol Buffers](https://developers.google.com/protocol-buffers/)
[JSON Adapter](https://github.com/casbin/json-adapter) | Stream | Casbin | Persistence for [JSON](https://www.json.org/)
[RQLite Adapter](https://github.com/edomosystems/rqlite-adapter) | SQL | [EDOMO Systems](https://github.com/edomosystems) | Persistence for [RQLite](https://github.com/rqlite/rqlite/)
[PostgreSQL Adapter](https://github.com/going/casbin-postgres-adapter) | SQL | [Going](https://github.com/going) | Persistence for [PostgreSQL](https://www.postgresql.org/)
[RethinkDB Adapter](https://github.com/adityapandey9/rethinkdb-adapter) | NoSQL | [@adityapandey9](https://github.com/adityapandey9) | Persistence for [RethinkDB](https://rethinkdb.com/)
[DynamoDB Adapter](https://github.com/HOOQTV/dynacasbin) | NoSQL | [HOOQ](https://github.com/HOOQTV) | Persistence for [Amazon DynamoDB](https://aws.amazon.com/dynamodb/)

### AutoSave

There is a feature called ``Auto-Save`` for adapters. When an adapter supports ``Auto-Save``, it means it can support adding a single policy rule to the storage, or removing a single policy rule from the storage. This is unlike ``SavePolicy()``, because the latter will delete all policy rules in the storage and save all policy rules from Casbin enforcer to the storage. So it may suffer performance issue when the number of policy rules is large.

When the adapter supports ``Auto-Save``, you can switch this option via ``Enforcer.EnableAutoSave()`` function. The option is enabled by default (if the adapter supports it).

**Note**:

1. The``Auto-Save`` feature is optional. An adapter can choose to implement it or not.
2. ``Auto-Save`` only works for a Casbin enforcer when the adapter the enforcer uses supports it.
3. Currently, the adapters that support ``Auto-Save`` are:

Adapter with ``Auto-Save`` | Type | Author | Description
----|------|----|----
[Xorm Adapter](https://github.com/casbin/xorm-adapter) | ORM | Casbin | MySQL, PostgreSQL, TiDB, SQLite, SQL Server, Oracle are supported by [Xorm](https://github.com/go-xorm/xorm/)
[Gorm Adapter](https://github.com/casbin/gorm-adapter) | ORM | Casbin | MySQL, PostgreSQL, Sqlite3, SQL Server are supported by [Gorm](https://github.com/jinzhu/gorm/)
[Beego ORM Adapter](https://github.com/casbin/beego-orm-adapter) | ORM | Casbin | MySQL, PostgreSQL, Sqlite3 are supported by [Beego ORM](https://beego.me/docs/mvc/model/overview.md)
[MongoDB Adapter](https://github.com/casbin/mongodb-adapter) | NoSQL | Casbin | Persistence for [MongoDB](https://www.mongodb.com)
[RQLite Adapter](https://github.com/edomosystems/rqlite-adapter) | SQL | [EDOMO Systems](https://github.com/edomosystems) | Persistence for [RQLite](https://github.com/rqlite/rqlite/)
[RethinkDB Adapter](https://github.com/adityapandey9/rethinkdb-adapter) | NoSQL | [@adityapandey9](https://github.com/adityapandey9) | Persistence for [RethinkDB](https://rethinkdb.com/)

Here's an example about how to use ``Auto-Save``:

```go
import (
    "github.com/casbin/casbin"
    "github.com/casbin/xorm-adapter"
    _ "github.com/go-sql-driver/mysql"
)

// By default, the AutoSave option is enabled for an enforcer.
a := xormadapter.NewAdapter("mysql", "mysql_username:mysql_password@tcp(127.0.0.1:3306)/")
e := casbin.NewEnforcer("examples/basic_model.conf", a)

// Disable the AutoSave option.
e.EnableAutoSave(false)

// Because AutoSave is disabled, the policy change only affects the policy in Casbin enforcer,
// it doesn't affect the policy in the storage.
e.AddPolicy(...)
e.RemovePolicy(...)

// Enable the AutoSave option.
e.EnableAutoSave(true)

// Because AutoSave is enabled, the policy change not only affects the policy in Casbin enforcer,
// but also affects the policy in the storage.
e.AddPolicy(...)
e.RemovePolicy(...)
```

For more examples, please see: https://github.com/casbin/xorm-adapter/blob/master/adapter_test.go

### How to write an adapter

All adapters should implement the [Adapter](https://github.com/casbin/casbin/blob/master/persist/adapter.go) interface by providing at least two mandatory methods:``LoadPolicy(model model.Model) error`` and ``SavePolicy(model model.Model) error``.

The other three functions are optional. They should be implemented if the adapter supports the ``Auto-Save`` feature.

Method | Type | Description
----|------|----
LoadPolicy() | mandatory | Load all policy rules from the storage
SavePolicy() | mandatory | Save all policy rules to the storage
AddPolicy() | optional | Add a policy rule to the storage
RemovePolicy() | optional | Remove a policy rule from the storage
RemoveFilteredPolicy() | optional | Remove policy rules that match the filter from the storage

**Note**: if an adapter doesn't support the ``Auto-Save`` feature, it should provide an empty implementation for the three optional functions. If you don't provide it, Golang compiler will complain errors. Here's an example:

```go
// AddPolicy adds a policy rule to the storage.
func (a *Adapter) AddPolicy(sec string, ptype string, rule []string) error {
	return errors.New("not implemented")
}

// RemovePolicy removes a policy rule from the storage.
func (a *Adapter) RemovePolicy(sec string, ptype string, rule []string) error {
	return errors.New("not implemented")
}

// RemoveFilteredPolicy removes policy rules that match the filter from the storage.
func (a *Adapter) RemoveFilteredPolicy(sec string, ptype string, fieldIndex int, fieldValues ...string) error {
	return errors.New("not implemented")
}
```

Casbin enforcer will ignore the ``not implemented`` error when calling these three optional functions.

#### Who is responsible to create the DB?

As a convention, the adapter should be able to automatically create a database named ``casbin``  if it doesn't exist and use it for policy storage. Please use the MySQL adapter as a reference implementation: https://github.com/casbin/mysql-adapter

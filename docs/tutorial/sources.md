# Sources

A Source describes where Sovereign can find the [Instances](/terminology/#instances) that will be used to create dynamic Envoy configuration.

Every source should have three fields: `type`, `scope`, and `config`.  

Read on to see examples of these fields being used.

## Types of Sources built-in to Sovereign

### Inline

Provides instances directly to sovereign via config.  
A good example for using an inline source is for instances that should be static or rarely changed.

```yaml
type: inline
scope: <default|clusters|listeners|...>
config:
  instances:
    - <Instance>
```

### File

Provides instances from a file on-disk, or a file over HTTP/HTTPS.  
Useful for retrieving instances dynamically from an API or a file that changes periodically.

```yaml
type: inline
scope: <default|clusters|listeners|...>
config:
  path: https+json://somewhere.com/api/instances.json
```

!!! note

    To add your own custom Source type, see [Custom Sources](/advanced/custom-sources/)

## Adding a single source

For this example we will use an `inline` source, since it's harder to demonstrate using a file/http source in documentation.

```yaml
sources: 
  - type: inline
    config:
      instances:
        # This data can have any structure, 
        # as long as it is a list of key:value mappings
        - name: instance-A-01
          address: 10.0.0.50
          group: A
          id: '01'

templates: {}
```

## Using multiple sources

Multiple Sources can be added, and will be combined into the same set of [Instances](/terminology/#instances).

```yaml
sources: 
  - type: inline
    config:
      instances:
        - name: instance-A-01
          address: 10.0.0.50
          group: A
          id: '01'
  - type: inline
    config:
      instances:
        - name: instance-B-01
          address: 10.100.24.10
          group: B
          id: '01'

templates: {}
```

When Sovereign starts up, it will read this configuration, and store the instances from all sources in memory.  
They'll look something like this:

```json
{
  "scopes": {
    "default": [
      {
        "name": "instance01",
        "address": "10.0.0.50",
        "group": "A"
      },
      {
        "name": "instance01",
        "address": "10.100.24.10",
        "group": "B"
      }
    ]
  }
}
```

There are two things to notice:

* Instances from both sources were placed together
* The Instances were stored in a "scope" called "default"

## Source scopes

If Sovereign had to use all of the instances from every source, for every resource type, there would be a lot of unnecessary 
conditional logic required to separate clusters from listeners, and so on.

For example the above instances only make sense to be used to generate either cluster or endpoint configuration.

For this reason, sources have an argument called `scope` which results in them being used only for particular resource types.

Changing the above source to be more appropriately scoped is as simple as follows:

```yaml
sources: 
  - type: inline
    scope: clusters  # <----------------
    config:
      instances:
        - name: instance01
          address: 10.0.0.50
          group: A
  - type: inline
    scope: clusters  # <----------------
    config:
      instances:
        - name: instance01
          address: 10.100.24.10
          group: B

templates: {}
```

Now that they are both scoped to `clusters`, they will only be available in the 
Cluster template that you configure for your Sovereign server.

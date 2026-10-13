# Installation

## Requirements

cbq requires the following:

* Adobe ColdFusion (ACF) 2018+ **OR** Lucee 5+
* ColdBox 6+

The different [Queue Providers](../configuration/providers/) each have their own requirements that are listed on their individual pages.

## Install via ForgeBox with CommandBox

cbq is installed via [ForgeBox](https://forgebox.io) with [CommandBox](https://www.ortussolutions.com/products/commandbox).  You can install the latest version using the command:

```shell
install cbq
```

To stay on the v7 release line, install a version constraint:

```shell
box install cbq@^7.0.0
```

Installing cbq does not apply database migrations. Before starting workers or dispatching batches, apply the migrations for the tables you use. See the [Upgrade Guide](../upgrade-guide.md#v6-to-v7) and [DBProvider setup](../configuration/providers/dbprovider.md#jobs-table-structure).

## Load Java Libraries

When using batches, cbq utilizes additional Java libraries included in the `lib/` folder. These need to be added to your Application's `javaSettings` in `Application.cfc`.

```cfscript
// Java Integration
this.javaSettings = {
    loadPaths: [ expandPath( "./modules/cbq/lib" ) ],
    loadColdFusionClassPath: true,
    reloadOnChange: false
};
```

{% hint style="info" %}
Feel free to use mappings to point to the cbq path, if needed.
{% endhint %}

Append the cbq library path if your application already defines `javaSettings.loadPaths`, and restart the application after changing it. A `ClassNotFoundException` for `com.fasterxml.uuid.Generators` means the bundled UUID library is missing from the application's classpath; changing the Java version alone does not add it. Use a Java version supported by your CFML engine.

## Additional Provider Installation Steps

The [Queue Provider ](../configuration/providers/)you use may have additional installation steps.  Check out the individual provider pages for more details.

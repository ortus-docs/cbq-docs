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

## Additional Provider Installation Steps

The [Queue Provider ](../configuration/providers/)you use may have additional installation steps.  Check out the individual provider pages for more details.

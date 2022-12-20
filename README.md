# LibModuleManager
A runtime/plugin-based, inter-plugin communication library for SourceMod.


## Version
vers 1.1.0a


## Purpose
To reduce the boilerplate needed to setup and create forwards and to simplify inter-plugin data sharing and control, all to facilitate good subplugin/module systems for complex server mods/systems.


## Features
* Global Forwards Manager - controlled and managed via Config file.
-- Let's you rapidly create and execute Global Forwards without needing to recompile plugins.

* Private Forward Managers - Any plugin using libmodulemanager can request one or more private forward managers, each individually controlled by a config file.
-- Let's you rapidly create and execute Private Forwards without needing to recompile plugins.

* Module/Plugin Managers - Any plugin using libmodulemanager, just like the Private Forward Managers, can request one or more Plugin Managers.

Each type of manager has different strengths to them.


* `SharedMap` - `StringMap` that has extra features to allow for very simple data sharing and data control between plugins.
-- Works on a system of properties.
-- Plugins that make properties, own those properties.
-- Plugins can optionally **[Un]Lock** (prevents other plugin's except `SharedMap` creator/owner plugin from deleting the property) and/or **[Un]Freeze** (prevents other plugin's except `SharedMap` creator/owner plugin from changing the property data).
-- `SharedMap`s are more type safe than normal `StringMap`s and allow you to get the length of an array or string of a property!


## How To Use

-- For creating Global Forwards, check out `configs/plugin_manager/global_fwds.cfg` to see an example config file on setting up forwards and modify/add forwards as needed.

-- For creating Private Forwards for a specific Private Forward Manager, check out `configs/plugin_manager/private_fwds_example.cfg` for an _example_ on Private Forward creation. **Remember that each private forward manager requires a cfg file for what private forwards you want, you can use the same cfg file if they will all share the same private forwards**.

-- For creating Module Managers, check out `configs/plugin_manager/module_manager_example.cfg` for an example on how to setup the operations for specific Module Managers.

For setting up data sharing and control, You need a core plugin that will setup the `SharedMap` that will be used with all the subplugins/modules that will communicate with the core. What you need to do is to first use the `OnLibraryAdded` forward, specifically checking for if the name is `LibModuleManager`. There we will create the `SharedMap` under a channel name of your own choosing!

Remember this information though:

* You can use `Lock`, `Unlock`, `Freeze`, and `Unfreeze` to control data security of each property.
* That only the owner plugin of a property can [un]lock and/or [un]freeze said property.
* The plugin that created the `SharedMap` can override any locked and/or frozen properties.

```cs
public void OnLibraryAdded(const char[] name) {
	if( StrEqual(name, "LibModuleManager") ) {
		SharedMap dict = SharedMap("my_plugin");
		
		/// create's a property,
		/// making the creator/owner plugin the owner of this property.
		dict.SetInt("Example property", 100);
		
		/// locking prevents other plugins from deleting this property [by accident].
		dict.Lock("Example property");
	}
}
```

With the core plugin having setup the `SharedMap` and the data channel, you're now ready to share data with subplugins/modules that will work with the core plugin.


Like the core plugin, you need to first use the `OnLibraryAdded` forward, specifically checking for if the name is `LibModuleManager`. However, with a subplugin/module, we need to use `PawnAwait` to create a Timer that'll _wait_ [checks and repeats basically] for the channel that will have the `SharedMap` we need from the core plugin. The function that you give to `PawnAwait` needs to return a `bool`. The timer will stop repeating and close when the function it uses returns `true`. A `false` return value will make the timer repeat again.

```cs
public void OnLibraryAdded(const char[] name) {
	if( StrEqual(name, "LibModuleManager") ) {
		/// `PawnAwait` is in plugin_utils.inc
		PawnAwait(AwaitChannel, 0.25, {0}, 0);
	}
}


static SharedMap g_shmap;

public bool AwaitChannel() {
	if( !LibModuleManager_ChannelExists("my_plugin") ) {
		return false;
	}
	g_shmap = SharedMap("my_plugin");
	return true;
}
```

It's important to remember that you can use the `SharedMap` map constructor to _retrieve_ a `SharedMap` from a specific channel. Inputing a different channel name [this includes typos] will open a new channel and create a new `SharedMap` for that plugin as the owner. This also allows a plugin dev to have specific and/or _tiered_ `SharedMap`s to be used as the data set for a group of subplugins/modules.

# esx_vehicleshop_craft_snippet
Craft License Plates With Cardealer Job

# IMPORTANT NOTE
For this snippet to work properly, you will need two resources installed.
* [JSFOUR License Plates](https://github.com/jonassvensson4/jsfour-licenseplate)
* [esx_vehicleshop](https://github.com/ESX-Org/esx_vehicleshop) and all required dependencies.

# CONFIG.LUA
* Find:
```
ResellVehicle = {
		Pos   = { x = 2091.81, y = 3402.73, z = 46.03 },
		Size  = { x = 3.0, y = 3.0, z = 1.0 },
		Type  = 1,
}
```
Put a comma after the _}_, press enter and past this:
```
Craft = {
		Pos   = { x = 2102.89, y = 3354.97, z = 46.6 },
		Size  = { x = 1.5, y = 1.5, z = 1.0 },
		Type  = 1,
}
```


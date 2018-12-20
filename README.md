# LICENSE FOR USE
* You may use this snippet and edit to your liking.  Please PR any improvements.
* This code may **NOT** be posted anywhere else.
* I will not help to install or install this for you.  If you don't know how to edit code, **DON'T**.
* I will delete any issues that do not show errors as I can not help without know what they are.

# esx_vehicleshop_craft_snippet
* Craft License Plates With Cardealer Job.
* Requires two steel items to craft one licensplate item.
* In some parts, it's easier to replace an entire section than it is to just try and find the small area to change.  Smart people will just be able to add in the proper code.

# KNOWN BUGS
* Marker is invisible until after a relog or when you start crafting.
* No other known bugs.

# IMPORTANT NOTE
For this snippet to work properly, you will need three resources installed first.
* [JSFOUR License Plates](https://github.com/jonassvensson4/jsfour-licenseplate)
* [esx_vehicleshop](https://github.com/ESX-Org/esx_vehicleshop) and all required dependencies.
* [esx_jobs](https://github.com/ESX-Org/esx_jobs)
* You also need to add the item "__steel__" to the "__items__" table of your database.


# ESX_JOBS
Navigate to client/jobs/

## MINER.LUA
Find:
```lua
	{
          name   = _U('m_diamond'),
          db_name= "diamond",
          max    = 50,
          add    = 1,
          drop   = 5
	}
```

Add a comma behind the __}__ then press enter and paste:
```lua
        {
          name   = _U('m_steel'),
          db_name= "steel",
          max    = 30,
          add    = 1,
          drop   = 4
        }
```
*I have steel set as more rare than diamonds, rarity is up to you*

# ESX_VEHICLESHOP
## CONFIG.LUA
Find:
```lua
ResellVehicle = {
		Pos   = { x = -44.630, y = -1080.738, z = 25.683 },
		Size  = { x = 3.0, y = 3.0, z = 1.0 },
		Type  = 1,
}
```
Put a comma after the **}**, press enter and past this:
```lua
Craft = {
		Pos   = { x = -39.80, y = -1088.24, z = 25.50 },
		Size  = { x = 1.5, y = 1.5, z = 1.0 },
		Type  = 1,
}
```
The above coords will not be the same as yours.  You will need to change these.
## SERVER/MAIN.LUA
Find:

```lua
function PayRent(d, h, m)
	MySQL.Async.fetchAll('SELECT * FROM rented_vehicles', {}, function (result)
		for i=1, #result, 1 do
			local xPlayer = ESX.GetPlayerFromIdentifier(result[i].owner)

			-- message player if connected
			if xPlayer ~= nil then
				xPlayer.removeAccountMoney('bank', result[i].rent_price)
				TriggerClientEvent('esx:showNotification', xPlayer.source, _U('paid_rental', ESX.Math.GroupDigits(result[i].rent_price)))
			else -- pay rent either way
				MySQL.Sync.execute('UPDATE users SET bank = bank - @bank WHERE identifier = @identifier',
				{
					['@bank']       = result[i].rent_price,
					['@identifier'] = result[i].owner
				})
			end

			TriggerEvent('esx_addonaccount:getSharedAccount', 'society_cardealer', function(account)
				account.addMoney(result[i].rent_price)
			end)
		end
	end)
end
```

After, paste:

```lua
local function Craft(source)
	SetTimeout(4000, function()

		if PlayersCrafting[source] == true then
			local xPlayer = ESX.GetPlayerFromId(source)
			local SteelQuantity = xPlayer.getInventoryItem('steel').count

			if SteelQuantity <= 0 then
				TriggerClientEvent('esx:showNotification', source, _U('not_enough_steel'))
			else
				xPlayer.removeInventoryItem('steel', 2)
				xPlayer.addInventoryItem('licenseplate', 1)
				Craft(source)
			end
		end

	end)
end

RegisterServerEvent('esx_vehicleshop:startCraft')
AddEventHandler('esx_sidealership:startCraft', function()
	local _source = source
	PlayersCrafting[_source] = true
	TriggerClientEvent('esx:showNotification', _source, _U('assembling_license_plate'))
	Craft(_source)
end)

RegisterServerEvent('esx_vehicleshop:stopCraft')
AddEventHandler('esx_sidealership:stopCraft', function()
	local _source = source
	PlayersCrafting[_source] = false
end)
```

## CLIENT/MAIN.LUA
find:
```lua
Citizen.CreateThread(function ()
	while ESX == nil do
		TriggerEvent('esx:getSharedObject', function(obj) ESX = obj end)
		Citizen.Wait(0)
	end

	Citizen.Wait(10000)

	ESX.TriggerServerCallback('esx_vehicleshop:getCategories', function (categories)
		Categories = categories
	end)

	ESX.TriggerServerCallback('esx_vehicleshop:getVehicles', function (vehicles)
		Vehicles = vehicles
	end)

	if Config.EnablePlayerManagement then
		if ESX.PlayerData.job.name == 'cardealer' then
			Config.Zones.ShopEntering.Type = 1

			if ESX.PlayerData.job.grade_name == 'boss' then
				Config.Zones.BossActions.Type = 1
			end

		else
			Config.Zones.ShopEntering.Type = -1
			Config.Zones.BossActions.Type  = -1
		end
	end
end)

RegisterNetEvent('esx:playerLoaded')
AddEventHandler('esx:playerLoaded', function(xPlayer)
	ESX.PlayerData = xPlayer

	if Config.EnablePlayerManagement then
		if ESX.PlayerData.job.name == 'cardealer' then
			Config.Zones.ShopEntering.Type = 1

			if ESX.PlayerData.job.grade_name == 'boss' then
				Config.Zones.BossActions.Type = 1
			end

		else
			Config.Zones.ShopEntering.Type = -1
			Config.Zones.BossActions.Type  = -1
		end
	end
end)
```

Replace with:
```lua
Citizen.CreateThread(function ()
	while ESX == nil do
		TriggerEvent('esx:getSharedObject', function(obj) ESX = obj end)
		Citizen.Wait(0)
	end

	Citizen.Wait(10000)

	ESX.TriggerServerCallback('esx_vehicleshop:getCategories', function (categories)
		Categories = categories
	end)

	ESX.TriggerServerCallback('esx_vehicleshop:getVehicles', function (vehicles)
		Vehicles = vehicles
	end)

	if Config.EnablePlayerManagement then
		if ESX.PlayerData.job.name == 'cardealer' then
			Config.Zones.ShopEntering.Type = 1

			if ESX.PlayerData.job.grade_name == 'boss' then
				Config.Zones.BossActions.Type = 1
			end

		elseif ESX.PlayerData.job.name == 'cardealer' then
			Config.Zones.Craft.Type = 1

		else
			Config.Zones.ShopEntering.Type = -1
			Config.Zones.Craft.Type 	   = -1
			Config.Zones.BossActions.Type  = -1
		end
	end
end)

RegisterNetEvent('esx:playerLoaded')
AddEventHandler('esx:playerLoaded', function(xPlayer)
	ESX.PlayerData = xPlayer

	if Config.EnablePlayerManagement then
		if ESX.PlayerData.job.name == 'cardealer' then
			Config.Zones.ShopEntering.Type = 1

			if ESX.PlayerData.job.grade_name == 'boss' then
				Config.Zones.BossActions.Type = 1
			end

		elseif ESX.PlayerData.job.name == 'cardealer' then
			Config.Zones.Craft.Type = 1

		else
			Config.Zones.ShopEntering.Type = -1
			Config.Zones.Craft.Type 	   = -1
			Config.Zones.BossActions.Type  = -1
		end
	end
end)
```
Find:
```lua
function OpenPutStocksMenu()
	ESX.TriggerServerCallback('esx_vehicleshop:getPlayerInventory', function (inventory)
		local elements = {}

		for i=1, #inventory.items, 1 do
			local item = inventory.items[i]

			if item.count > 0 then
				table.insert(elements, {
					label = item.label .. ' x' .. item.count,
					type = 'item_standard',
					value = item.name
				})
			end
		end

		ESX.UI.Menu.Open('default', GetCurrentResourceName(), 'stocks_menu',
		{
			title    = _U('inventory'),
			align    = 'top-left',
			elements = elements
		}, function (data, menu)
			local itemName = data.current.value

			ESX.UI.Menu.Open('dialog', GetCurrentResourceName(), 'stocks_menu_put_item_count', {
				title = _U('amount')
			}, function (data2, menu2)
				local count = tonumber(data2.value)

				if count == nil then
					ESX.ShowNotification(_U('quantity_invalid'))
				else
					TriggerServerEvent('esx_vehicleshop:putStockItems', itemName, count)
					menu2.close()
					menu.close()
					OpenPutStocksMenu()
				end

			end, function (data2, menu2)
				menu2.close()
			end)

		end, function (data, menu)
			menu.close()
		end)
	end)
end
```

Below, paste:
```lua
function OpenLicensePlateCraftMenu()
	if Config.EnablePlayerManagement then
		if ESX.PlayerData.job.name == 'cardealer' then
			Config.Zones.Craft.Type = 1
		else
			Config.Zones.Craft.Type = -1
		end
		
		local elements = {
			{label = _U('licenseplate'),  value = 'license_plate'}
		}

		ESX.UI.Menu.CloseAll()

		ESX.UI.Menu.Open('default', GetCurrentResourceName(), 'license_plate_craft', {
			title    = _U('craft'),
			align    = 'top-left',
			elements = elements
		}, function(data, menu)
			if data.current.value == 'license_plate' then
				menu.close()
				TriggerServerEvent('esx_vehicleshop:startCraft')
			end

		end, function(data, menu)
			menu.close()

			CurrentAction     = 'license_plate_menu'
			CurrentActionMsg  = _U('craft_menu')
			CurrentActionData = {}
		end)

	else
		ESX.ShowNotification(_U('not_experienced_enough'))
	end
end
```

Find:
```lua
RegisterNetEvent('esx:setJob')
AddEventHandler('esx:setJob', function (job)
	ESX.PlayerData.job = job

	if Config.EnablePlayerManagement then
		if ESX.PlayerData.job.name == 'cardealer' then
			Config.Zones.ShopEntering.Type = 1

			if ESX.PlayerData.job.grade_name == 'boss' then
				Config.Zones.BossActions.Type = 1
			end

		else
			Config.Zones.ShopEntering.Type = -1
			Config.Zones.BossActions.Type  = -1
		end
	end
end)
```

Replace with:
```lua
RegisterNetEvent('esx:setJob')
AddEventHandler('esx:setJob', function (job)
	ESX.PlayerData.job = job

	if Config.EnablePlayerManagement then
		if ESX.PlayerData.job.name == 'cardealer' then
			Config.Zones.ShopEntering.Type = 1

			if ESX.PlayerData.job.grade_name == 'boss' then
				Config.Zones.BossActions.Type = 1
			end
		elseif ESX.PlayerData.job.name == 'cardealer' then
			   Config.Zones.Craft.Type = 1
		else
			Config.Zones.ShopEntering.Type = -1
			Config.Zones.Craft.Type 	   = -1
			Config.Zones.BossActions.Type  = -1
		end
	end
end)
```

Find:
```lua
	elseif zone == 'BossActions' and Config.EnablePlayerManagement and ESX.PlayerData.job ~= nil and ESX.PlayerData.job.name == 'cardealer' and ESX.PlayerData.job.grade_name == 'boss' then

		CurrentAction     = 'boss_actions_menu'
		CurrentActionMsg  = _U('shop_menu')
		CurrentActionData = {}

	end
```

Above, paste:
```lua
	elseif zone == 'Craft' and Config.EnablePlayerManagement and ESX.PlayerData.job ~= nil and ESX.PlayerData.job.name == 'cardealer' then
		CurrentAction     = 'license_plate_craft_menu'
		CurrentActionMsg  = _U('craft_menu')
		CurrentActionData = {}
```

Find:
```lua
AddEventHandler('esx_vehicleshop:hasExitedMarker', function (zone)
	if not IsInShopMenu then
		ESX.UI.Menu.CloseAll()
	end

	CurrentAction = nil
end)
```

Replace with:
```lua
AddEventHandler('esx_vehicleshop:hasExitedMarker', function (zone)
	if not IsInShopMenu then
		ESX.UI.Menu.CloseAll()
	-- Craft License Plate
	elseif zone == 'Craft' then
		TriggerServerEvent('esx_vehicleshop:stopCraft')
	end

	CurrentAction = nil
end)
```

Find:
```lua
Citizen.CreateThread(function()
	while true do
		Citizen.Wait(10)

		if CurrentAction == nil then
			Citizen.Wait(500)
		else

			ESX.ShowHelpNotification(CurrentActionMsg)

			if IsControlJustReleased(0, Keys['E']) then
				if CurrentAction == 'shop_menu' then
					OpenShopMenu()
				elseif CurrentAction == 'reseller_menu' then
					OpenResellerMenu()
				elseif CurrentAction == 'give_back_vehicle' then

					ESX.TriggerServerCallback('esx_vehicleshop:giveBackVehicle', function(isRentedVehicle)
						if isRentedVehicle then
							ESX.Game.DeleteVehicle(CurrentActionData.vehicle)
							ESX.ShowNotification(_U('delivered'))
						else
							ESX.ShowNotification(_U('not_rental'))
						end
					end, ESX.Math.Trim(GetVehicleNumberPlateText(CurrentActionData.vehicle)))

				elseif CurrentAction == 'resell_vehicle' then

					ESX.TriggerServerCallback('esx_vehicleshop:resellVehicle', function(vehicleSold)

						if vehicleSold then
							ESX.Game.DeleteVehicle(CurrentActionData.vehicle)
							ESX.ShowNotification(_U('vehicle_sold_for', CurrentActionData.label, ESX.Math.GroupDigits(CurrentActionData.price)))
						else
							ESX.ShowNotification(_U('not_yours'))
						end

					end, CurrentActionData.plate, CurrentActionData.model)

				elseif CurrentAction == 'boss_actions_menu' then
					OpenBossActionsMenu()
				end

				CurrentAction = nil
			end
		end
	end
end)
```

Replace with:
```lua
Citizen.CreateThread(function()
	while true do
		Citizen.Wait(10)

		if CurrentAction == nil then
			Citizen.Wait(500)
		else

			ESX.ShowHelpNotification(CurrentActionMsg)

			if IsControlJustReleased(0, Keys['E']) then
				if CurrentAction == 'shop_menu' then
					OpenShopMenu()
				elseif CurrentAction == 'reseller_menu' then
					OpenResellerMenu()
				elseif CurrentAction == 'give_back_vehicle' then

					ESX.TriggerServerCallback('esx_vehicleshop:giveBackVehicle', function(isRentedVehicle)
						if isRentedVehicle then
							ESX.Game.DeleteVehicle(CurrentActionData.vehicle)
							ESX.ShowNotification(_U('delivered'))
						else
							ESX.ShowNotification(_U('not_rental'))
						end
					end, ESX.Math.Trim(GetVehicleNumberPlateText(CurrentActionData.vehicle)))

				elseif CurrentAction == 'resell_vehicle' then

					ESX.TriggerServerCallback('esx_vehicleshop:resellVehicle', function(vehicleSold)

						if vehicleSold then
							ESX.Game.DeleteVehicle(CurrentActionData.vehicle)
							ESX.ShowNotification(_U('vehicle_sold_for', CurrentActionData.label, ESX.Math.GroupDigits(CurrentActionData.price)))
						else
							ESX.ShowNotification(_U('not_yours'))
						end

					end, CurrentActionData.plate, CurrentActionData.model)

				elseif CurrentAction == 'boss_actions_menu' then
					OpenBossActionsMenu()

				elseif CurrentAction == 'license_plate_craft_menu' then
					OpenLicensePlateCraftMenu()
				end

				CurrentAction = nil
			end
		end
	end
end)
```

**DONE**

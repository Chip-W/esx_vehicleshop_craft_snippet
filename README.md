# esx_vehicleshop_craft_snippet
Craft License Plates With Cardealer Job

# IMPORTANT NOTE
For this snippet to work properly, you will need three resources installed.
* [JSFOUR License Plates](https://github.com/jonassvensson4/jsfour-licenseplate)
* [esx_vehicleshop](https://github.com/ESX-Org/esx_vehicleshop) and all required dependencies.
* [esx_jobs](https://github.com/ESX-Org/esx_jobs)
* You also need to add the item "__steel__" to your "__items__" table of your database.


# ESX_JOBS

# ESX_VEHICLESHOP
## CONFIG.LUA
Find:
```
ResellVehicle = {
		Pos   = { x = 2091.81, y = 3402.73, z = 46.03 },
		Size  = { x = 3.0, y = 3.0, z = 1.0 },
		Type  = 1,
}
```
Put a comma after the **}**, press enter and past this:
```
Craft = {
		Pos   = { x = 2102.89, y = 3354.97, z = 46.6 },
		Size  = { x = 1.5, y = 1.5, z = 1.0 },
		Type  = 1,
}
```
<br>
<br>
## SERVER/MAIN.LUA
Find
```
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
```
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

RegisterServerEvent('esx_sidealership:startCraft')
AddEventHandler('esx_sidealership:startCraft', function()
	local _source = source
	PlayersCrafting[_source] = true
	TriggerClientEvent('esx:showNotification', _source, _U('assembling_license_plate'))
	Craft(_source)
end)

RegisterServerEvent('esx_sidealership:stopCraft')
AddEventHandler('esx_sidealership:stopCraft', function()
	local _source = source
	PlayersCrafting[_source] = false
end)
```

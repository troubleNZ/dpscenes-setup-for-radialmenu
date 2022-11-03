![Commits](https://img.shields.io/github/last-commit/troublenz/trbl-LicensePed?style=plastic) 
![OpenIssues](https://img.shields.io/github/issues/troublenz/trbl-LicensePed?style=plastic) 
![Contributors](https://img.shields.io/github/contributors/troublenz/trbl-LicensePed?color=aqua&style=plastic) 
![Size](https://img.shields.io/github/repo-size/troublenz/trbl-LicensePed?color=aqua&style=plastic) 
![Languages](https://img.shields.io/github/languages/top/troublenz/trbl-LicensePed?color=aqua&style=plastic)
# dpscenes-setup-for-radialmenu
### Add dpscene commands to the qb-radialmenu

![General](https://cdn.discordapp.com/attachments/866268698359496743/988319144183222282/unknown.png)

Dependencies:
- [qb-radialmenu](https://github.com/qbcore-framework/qb-radialmenu)
- [qb-input](https://github.com/qbcore-framework/qb-input)
- [dpscenes](https://github.com/andristum/dpscenes)

## Step 1 - add to qb-radialmenu config

Config.lua
```lua
[5] = {
        id = 'scenes',
        title = 'Scenes',
        icon = 'comment-dots',
        items = {
            {
                id = 'newscene',
                title = 'Create New',
                icon = 'comment-dots',
                type = 'client',
                event = 'trbl:client:dpscene:keyboardentry',
                shouldClose = true
            
            }, {
                id = 'removescene',
                title = 'Remove',
                icon = 'cut',
                type = 'client',
                event = 'trbl:client:dpscene:sceneremove',
                shouldClose = true
            }, {
                id = 'copyscene',
                title = 'Copy Scene',
                icon = 'copy',
                type = 'client',
                event = 'trbl:client:dpscene:scenecopy',
                shouldClose = true
            }, {
                id = 'movescene',
                title = 'Move Scene',
                icon = 'redo-alt',
                type = 'client',
                event = 'trbl:client:dpscene:scenemove',
                shouldClose = true
            }
        }
    },
```

## Step 2 - add to qb-radialmenu/client/main.lua
```lua
-- dpscenes triggered via radialmenu
-- player entry field
RegisterNetEvent('trbl:client:dpscene:keyboardentry')
AddEventHandler('trbl:client:dpscene:keyboardentry',function()

local dialog = exports['qb-input']:ShowInput({
	header = "Enter your Text",
	submitText = "Submit",
	inputs = {{ type = 'text', isRequired = true, name = 'SceneText', text = 'Text' }}})
	if dialog then
		--print(dialog.SceneText)	--debug
		TriggerServerEvent('trbl:dpscene:server:SceneInput', dialog.SceneText)			--  the event is in the dpscene server lua
    end
end)
```

## Step 3 - add to dpscenes/Client/Scenes.lua

```lua

-- triggered via radialmenu
RegisterNetEvent('trbl:client:dpscene:sceneremove')
AddEventHandler('trbl:client:dpscene:sceneremove',function()
	local Pos = GetEntityCoords(PlayerPedId())
	local Delete = {Id = 0, Distance = 3}
	for k,v in pairs(Scenes) do
		local Dis = Distance(Pos, v.Location)
		if Dis < Delete.Distance then
			Delete = {Id = k,Distance = Dis}
		end
	end
	if Delete.Id ~= 0 then
		TriggerServerEvent("Scene:AttemptDelete", Delete.Id)
	end
end)

-- triggered via radialmenu
RegisterNetEvent('trbl:client:dpscene:scenecopy')
AddEventHandler('trbl:client:dpscene:scenecopy',function()
	local Pos = GetEntityCoords(PlayerPedId())
	local Copy = {Id = 0, Distance = 3}
	for k,v in pairs(Scenes) do
		local Dis = Distance(Pos, v.Location)
		if Dis < Copy.Distance then
			Copy = {Id = k,Distance = Dis}
		end
	end
	if Copy.Id ~= 0 then
		TriggerServerEvent("Scene:AttemptCopy", Copy.Id)
	else
		Chat(Lang("CouldntFindCopy"))
	end
end)

-- triggered via radialmenu
RegisterNetEvent('trbl:client:dpscene:scenemove')
AddEventHandler('trbl:client:dpscene:scenemove',function()
	if Scene.Placing then
		Chat("Cant do this right now.")
		return
	end
	local Pos = GetEntityCoords(PlayerPedId())
	local Move = {Id = 0, Distance = 3}
	for k,v in pairs(Scenes) do
		local Dis = Distance(Pos, v.Location)
		if Dis < Move.Distance then
			Move = {Id = k,Distance = Dis}
		end
	end
	if Move.Id ~= 0 then
		StartMoveScene(Scenes[Move.Id], Move.Id)
	else
		Chat(Lang("CouldntFindMove"))
	end

end)
```

## Step 4 - add to dpscenes/Server/Scenes.lua
```
RegisterNetEvent('trbl:dpscene:server:SceneInput')
AddEventHandler('trbl:dpscene:server:SceneInput',function(Arguments)
	local _source = source
	if not Scenes.Cooldowns[_source] then
		Scenes.Cooldowns[_source] = 1
		local Length = string.len(Arguments)
		if Length <= 280 and Length > 3 then
			TriggerClientEvent("Scene:Create", _source, Arguments)
		else
			Chat(_source, Lang("InvalidText"))
		end
	else
		Chat(_source, Lang("CantRightNow"))
	end
end)
```

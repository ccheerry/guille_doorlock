# guille_doorlock
A simple resource to create in-game doorlocks.
Any problem feel free to ask in the guillerp discord, I will be pleasured to help you, You can also feel free to contact my DMs (CherryozZ#5357)

### [Discord](https://discord.gg/eBpmkW6e5j)

# License
This project does not contain a license, therefore you are not allowed to add one and claim it as yours. You are not allowed to sell this nor re-distribute it. You are not allowed to change/add a license. If you want to modify or make an agreement, you can contact me. Pull requests are accepted as long as they do not contain breaking changes. You can read more [HERE](https://opensource.stackexchange.com/questions/1720/what-can-i-assume-if-a-publicly-published-project-has-no-license) 

### Requirements

- QBCore
- qb-menu-default (https://github.com/CherryozZ/qb-menu-default)
- qb-menu-dialog  (https://github.com/CherryozZ/qb-menu-dialog)

### Installation
1) Download from the releases tab.
2) Move the resource into your resources folder.
3) Add `ensure guille_doorlock` in your server.cfg
4) Apply the following modifications on your qb-core -->
- Add this on top of the qb-core/client/functions.lua, below QBCore.RequestId = 0

```lua
QBCore.TimeoutCallbacks          = {}
```
- Then add this on the bottom of the qb-core/client/functions.lua

```lua
QBCore.Table = {}

function QBCore.Table.SizeOf(t)
	local count = 0

	for _,_ in pairs(t) do
		count = count + 1
	end

	return count
end

Citizen.CreateThread(function()
	while true do
		local wait = 500
		local currTime = GetGameTimer()
		if #QBCore.TimeoutCallbacks > 0 then
			wait = 0
			for i=1, #QBCore.TimeoutCallbacks, 1 do
				if QBCore.TimeoutCallbacks[i] then
					if currTime >= QBCore.TimeoutCallbacks[i].time then
						QBCore.TimeoutCallbacks[i].cb()
						QBCore.TimeoutCallbacks[i] = nil
					end
				end
			end
		end
		Wait(wait)
	end
end)

QBCore.Math = {}

QBCore.Math.Round = function(value, numDecimalPlaces)
	if numDecimalPlaces then
		local power = 10^numDecimalPlaces
		return math.floor((value * power) + 0.5) / (power)
	else
		return math.floor(value + 0.5)
	end
end

QBCore.Math.Trim = function(value)
	if value then
		return (string.gsub(value, "^%s*(.-)%s*$", "%1"))
	else
		return nil
	end
end

QBCore.SetTimeout = function(msec, cb)
	table.insert(QBCore.TimeoutCallbacks, {
		time = GetGameTimer() + msec,
		cb   = cb
	})
	return #QBCore.TimeoutCallbacks
end

QBCore.ClearTimeout = function(i)
	QBCore.TimeoutCallbacks[i] = nil
end

QBCore.UI                        = {}
QBCore.UI.Menu                   = {}
QBCore.UI.Menu.RegisteredTypes   = {}
QBCore.UI.Menu.Opened            = {}


QBCore.UI.Menu.RegisterType = function(type, open, close)
	QBCore.UI.Menu.RegisteredTypes[type] = {
		open   = open,
		close  = close
	}
end

QBCore.UI.Menu.Open = function(type, namespace, name, data, submit, cancel, change, close)
	local menu = {}

	menu.type      = type
	menu.namespace = namespace
	menu.name      = name
	menu.data      = data
	menu.submit    = submit
	menu.cancel    = cancel
	menu.change    = change

	menu.close = function()

		QBCore.UI.Menu.RegisteredTypes[type].close(namespace, name)

		for i=1, #QBCore.UI.Menu.Opened, 1 do
			if QBCore.UI.Menu.Opened[i] then
				if QBCore.UI.Menu.Opened[i].type == type and QBCore.UI.Menu.Opened[i].namespace == namespace and QBCore.UI.Menu.Opened[i].name == name then
					QBCore.UI.Menu.Opened[i] = nil
				end
			end
		end

		if close then
			close()
		end

	end

	menu.update = function(query, newData)

		for i=1, #menu.data.elements, 1 do
			local match = true

			for k,v in pairs(query) do
				if menu.data.elements[i][k] ~= v then
					match = false
				end
			end

			if match then
				for k,v in pairs(newData) do
					menu.data.elements[i][k] = v
				end
			end
		end

	end

	menu.refresh = function()
		QBCore.UI.Menu.RegisteredTypes[type].open(namespace, name, menu.data)
	end

	menu.setElement = function(i, key, val)
		menu.data.elements[i][key] = val
	end

	menu.setElements = function(newElements)
		menu.data.elements = newElements
	end

	menu.setTitle = function(val)
		menu.data.title = val
	end

	menu.removeElement = function(query)
		for i=1, #menu.data.elements, 1 do
			for k,v in pairs(query) do
				if menu.data.elements[i] then
					if menu.data.elements[i][k] == v then
						table.remove(menu.data.elements, i)
						break
					end
				end

			end
		end
	end

	table.insert(QBCore.UI.Menu.Opened, menu)
	QBCore.UI.Menu.RegisteredTypes[type].open(namespace, name, data)

	return menu
end

QBCore.UI.Menu.Close = function(type, namespace, name)
	for i=1, #QBCore.UI.Menu.Opened, 1 do
		if QBCore.UI.Menu.Opened[i] then
			if QBCore.UI.Menu.Opened[i].type == type and QBCore.UI.Menu.Opened[i].namespace == namespace and QBCore.UI.Menu.Opened[i].name == name then
				QBCore.UI.Menu.Opened[i].close()
				QBCore.UI.Menu.Opened[i] = nil
			end
		end
	end
end

QBCore.UI.Menu.CloseAll = function()
	for i=1, #QBCore.UI.Menu.Opened, 1 do
		if QBCore.UI.Menu.Opened[i] then
			QBCore.UI.Menu.Opened[i].close()
			QBCore.UI.Menu.Opened[i] = nil
		end
	end
end

QBCore.UI.Menu.GetOpened = function(type, namespace, name)
	for i=1, #QBCore.UI.Menu.Opened, 1 do
		if QBCore.UI.Menu.Opened[i] then
			if QBCore.UI.Menu.Opened[i].type == type and QBCore.UI.Menu.Opened[i].namespace == namespace and QBCore.UI.Menu.Opened[i].name == name then
				return QBCore.UI.Menu.Opened[i]
			end
		end
	end
end

QBCore.UI.Menu.GetOpenedMenus = function()
	return QBCore.UI.Menu.Opened
end

QBCore.UI.Menu.IsOpen = function(type, namespace, name)
	return QBCore.UI.Menu.GetOpened(type, namespace, name) ~= nil
end

```

5) Add to your qb-core/client/wrapper.lua
6) Add to your qb-core/html/js/wrapper.js

https://github.com/CherryozZ/qb-wrapper

7) Modify your qb-core/fxmanifest.lua to

```lua
fx_version 'cerulean'
game 'gta5'

description 'QB-Core'
version '1.0.0'

shared_scripts { 
	'import.lua',
	'config.lua',
	'shared.lua'
}

client_scripts {
	'client/*.lua'
}

server_scripts {
	'server/*.lua'
}

ui_page {
	'html/ui.html'
}

lua54 'yes'

files {
	'html/ui.html',
	'html/css/*.css',
	'html/js/*.js'
}
```


8) Start your server.

## Documentation

- Creating a door

Use the command /door to create a door, if you are creating a double door be sure of aiming to a correct spot before confirm to be sure that you setted correctly the door.

- Deleting a door

Use the command /deletedoor aiming to the door, it may not work at first try.

- Export error

You are using a pretty old QBCore version, or leaked qbus files (illegal).

- "<" error?

Update your artifacts because the resource uses lua 5.4 .

# Credits

- The whole resource, I only converted it for QBCore (https://github.com/guillerp8/guille_doorlock)
- Readme inspired by Bombay (Entity Evolution)

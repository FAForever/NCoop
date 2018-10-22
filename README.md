# Nomads Coop
A mod that's required to run the Nomads coop missions in the Forged Alliance Forever Client.

It's supposed to be run as a featured mod, similarly to the coop mod.

# Setting up for testing
This is how to be able to run the mod with the nomads coop campaign, its good for testing the missions. To develop the mod itself see *Setting up for development*

1. Download this repository, and pack its contents into a .zip file
2. Rename the zip file to `nomads_coop.nmd`
3. Place the file into your gamedata folder at `C:\ProgramData\FAForever\gamedata`
4. Get a modified version of the `init_coop.lua` file - see below
5. Place that into `C:\ProgramData\FAForever\bin`, overwriting anything already present
6. Set the `init_coop.lua` file to read-only
7. Ensure you have a nomads coop campaign mission to test, they can be downloaded from the [nomads missions repository]: https://github.com/FAForever/NomadMissions or elsewhere.
8. Host a coop game through FAF - if everything has been done correctly, an error that a file could not be downloaded should appear. This is because you set the modified `init_coop.lua` to read only.
9. Change the map to a nomads supported map in the lobby. Currently only the nomads campaign is supported.
10. This _will_ cause desyncs if not everyone in your game has done this procedure.
11. To disable the mod, rename `init_coop.lua` to something else or clear the read only flag, so the next game it will be updated and not load the nomads mod anymore.


## Modifying init_coop.lua for testing
1. Create or open `init_coop.lua` inside `C:\ProgramData\FAForever\bin`
Paste the following:
```lua
sc_path = ""
dofile(InitFileDir .. '\\..\\fa_path.lua')

path = {}

whitelist =
{
    "effects.nx2",
    "env.nx2",
    "etc.nx2",
    "loc.nx2",
    "lua.nx2",
    "meshes.nx2",
    "mods.nx2",
    "modules.nx2",
    "projectiles.nx2",
    "schook.nx2",
    "textures.nx2",
    "units.nx2",
    "murderparty.nxt",
    "labwars.nxt",
    "units.scd",
    "textures.scd",
    "skins.scd",
    "schook.scd",
    "props.scd",
    "projectiles.scd",
    "objects.scd",
    "moholua.scd",
    "mohodata.scd",
    "mods.scd",
    "meshes.scd",
    "lua.scd",
    "loc_us.scd",
    "loc_es.scd",
    "loc_fr.scd",
    "loc_it.scd",
    "loc_de.scd",
    "loc_ru.scd",
    "env.scd",
    "effects.scd",
    "editor.scd",
    "ambience.scd",
    "advanced strategic icons.nxt",
    "lobbymanager.scd",
    "texturepack.nxt",
    "sc_music.scd"
}

local function mount_dir(dir, mountpoint)
    table.insert(path, { dir = dir, mountpoint = mountpoint } )
end


local function mount_contents(dir, mountpoint)
    LOG('checking ' .. dir)
    for _,entry in io.dir(dir .. '\\*') do
        if entry != '.' and entry != '..' then
        local mp = string.lower(entry)
        local safe = true
        mp = string.gsub(mp, '[.]scd$', '')
        mp = string.gsub(mp, '[.]zip$', '')
        mount_dir(dir .. '\\' .. entry, mountpoint .. '/' .. mp)
        end
    end
end

local function mount_dir_with_glob(dir, glob, mountpoint)
    sorted = {}
    LOG('checking ' .. dir .. glob)
    for _,entry in io.dir(dir .. glob) do
        if entry != '.' and entry != '..' then
            table.insert(sorted, dir .. entry)
        end
    end
    table.sort(sorted)
    table.foreach(sorted, function(k,v) mount_dir(v, mountpoint) end)
end

local function mount_dir_with_whitelist(dir, glob, mountpoint)
    sorted = {}
    LOG('checking ' .. dir .. glob)
    for _,entry in io.dir(dir .. glob) do
        if entry != '.' and entry != '..' then
            local mp = string.lower(entry)
            local notsafe = true
            for i, white in whitelist do
                notsafe = notsafe and (string.find(mp, white, 1) == nil)
            end
            if notsafe then
                LOG('not safe ' .. dir .. entry)
            else
                table.insert(sorted, dir .. entry)
            end
        end
    end
    table.sort(sorted)
    table.foreach(sorted, function(k,v) mount_dir(v, mountpoint) end)
end

local function mount_dir_with_blacklist(dir, glob, mountpoint)
    sorted = {}
    LOG('checking ' .. dir .. glob)
    for _,entry in io.dir(dir .. glob) do
        if entry != '.' and entry != '..' then
            local mp = string.lower(entry)
            local safe = true
            for i, black in blacklist do
                safe = safe and (string.find(mp, black, 1) == nil)
            end
            if safe then
                table.insert(sorted, dir .. entry)
            else
                LOG('not safe ' .. dir .. entry)
            end
        end
    end
    table.sort(sorted)
    table.foreach(sorted, function(k,v) mount_dir(v, mountpoint) end)
end

local function mount_map_dir(dir, glob, mountpoint)
    LOG('mounting maps from: '..dir)
    mount_contents(dir, mountpoint)

    for _, map in io.dir(dir..glob) do
        for _, folder in io.dir(dir..'\\'..map..'\\**') do
            if folder == 'movies' then
                LOG('Found map movies in: '..map)
                mount_dir(dir..map..'\\movies', '/movies')
            elseif folder == 'sounds' then
                LOG('Found map sounds in: '..map)
                mount_dir(dir..map..'\\sounds', '/sounds')
            end
        end
    end
end

local function clear_cache()
    local dir = SHGetFolderPath('LOCAL_APPDATA') .. 'Gas Powered Games\\Supreme Commander Forged Alliance\\cache\\'
    LOG('Clearing cached shader files in: ' .. dir)
    for _,file in io.dir(dir .. '**') do
        if string.find(file, 'mesh') then
            os.remove(dir .. file)
        end
    end
end

-- Clear the shader
clear_cache()

-- mount maps
mount_map_dir(SHGetFolderPath('PERSONAL') .. 'My Games\\Gas Powered Games\\Supreme Commander Forged Alliance\\maps\\', '**', '/maps')
mount_map_dir(InitFileDir .. '\\..\\user\\My Games\\Gas Powered Games\\Supreme Commander Forged Alliance\\maps\\', '**', '/maps')

-- mount mods
mount_contents(SHGetFolderPath('PERSONAL') .. 'My Games\\Gas Powered Games\\Supreme Commander Forged Alliance\\mods', '/mods')
mount_contents(InitFileDir .. '\\..\\user\\My Games\\Gas Powered Games\\Supreme Commander Forged Alliance\\mods', '/mods')

mount_dir(InitFileDir .. '\\..\\gamedata\\nomads_coop.nmd', '/')
mount_dir(InitFileDir..'\\..\\gamedata\\units.nmd', '/')
mount_dir(InitFileDir..'\\..\\gamedata\\textures.nmd', '/')
mount_dir(InitFileDir..'\\..\\gamedata\\sounds.nmd', '/')
mount_dir(InitFileDir..'\\..\\gamedata\\effects.nmd', '/')
mount_dir(InitFileDir..'\\..\\gamedata\\env.nmd', '/')
mount_dir(InitFileDir..'\\..\\gamedata\\nomadhook.nmd', '/')
mount_dir(InitFileDir..'\\..\\gamedata\\lua.nmd', '/')
mount_dir(InitFileDir..'\\..\\gamedata\\projectiles.nmd', '/')
mount_dir(InitFileDir..'\\..\\gamedata\\loc.nmd', '/')
mount_dir(InitFileDir..'\\..\\gamedata\\meshes.nmd', '/')

mount_dir(InitFileDir .. '\\..\\gamedata\\*.cop', '/')
mount_dir_with_whitelist(InitFileDir .. '\\..\\gamedata\\', '*.nxt', '/')
mount_dir_with_whitelist(InitFileDir .. '\\..\\gamedata\\', '*.nx2', '/')

mount_dir_with_whitelist(fa_path .. '\\gamedata\\', '*.scd', '/')
mount_dir(fa_path, '/')
mount_dir(InitFileDir .. '\\..\\movies', '/movies')

mount_dir_with_glob(InitFileDir .. '\\..\\gamedata\\', '*_VO.nx2', '/')



hook = {
    '/schook',
    '/mods/coop/hook',
    '/sounds',
    '/nomadhook',
}



protocols = {
    'http',
    'https',
    'mailto',
    'ventrilo',
    'teamspeak',
    'daap',
    'im',
}
```

# Setting up for development



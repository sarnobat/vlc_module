# vlc_module

## plugin in c programming

I still haven't figured out how to do this

## lua

It's easy but the documentation is so confusing or absent it's not funny. All you need to do is this:

* copy `youtube.lua` (not youtube.luac) from the source tree to `/Applications/VLC.app/Contents/MacOS/share/lua/playlist/youtube.lua` (no need to compile). You can rename it or not, up to you.
* add your logic to the `probe()` function
* run VLC (if you want to check if the script got loaded on startup, run `/Applications/VLC.app/Contents/MacOS/VLC --verbose 2` (note it writes to stderr, not stdout)
* You should see that it gets invoked either in the console output or in the actual output file.

### now_playing.txt logic

```
local hex_to_char = function(x)
  return string.char(tonumber(x, 16))
end

-- This must be defined before it is called.
local unescape = function(url)
  return url:gsub("%%(%x%x)", hex_to_char)
end

-- Probe function.
function probe()

	vlc.msg.info( "SRIDHAR probe 1: " .. vlc.path)
	file = io.open("/tmp/now_playing_vlc.txt", "a")
	file:write(unescape(vlc.path) .. "\n")
```

More info: https://wiki.videolan.org/Documentation:Building_Lua_Playlist_Scripts/#Simple_Examples

### Notes
* Don't bother with an "extension" script (https://translatedby.com/you/instructions-to-code-your-own-vlc-lua-scripts/original/), they only get launched via the menu.
* Don't bother with an "interface" script since you have to use that instead of an existing interface and change how you launch the app
* "Playlist" is the one you want.
* The terminology is confusing: script and extension are not the same thing, and I'm not sure what a module is. Actually I think the lua module that allows scripting is just one of many modules. Modules probably have to be written in C. I think a plugin and module are the same thing.

### Call graph

```
  * frame #0: 0x00000001069f9213 liblua_plugin.dylib`vlclua_input_item_get_internal(L=0x00000001003392f0) at input.c:276
    frame #1: 0x00000001069f8b10 liblua_plugin.dylib`vlclua_input_item_metas(L=0x00000001003392f0) at input.c:320
    frame #2: 0x0000000106a0a98d liblua_plugin.dylib`luaD_precall + 589
    frame #3: 0x0000000106a172ac liblua_plugin.dylib`luaV_execute + 2764
    frame #4: 0x0000000106a0b042 liblua_plugin.dylib`luaD_call + 98
    frame #5: 0x0000000106a0a485 liblua_plugin.dylib`luaD_rawrunprotected + 85
    frame #6: 0x0000000106a0b35a liblua_plugin.dylib`luaD_pcall + 58
    frame #7: 0x0000000106a05eca liblua_plugin.dylib`lua_pcall + 282
    frame #8: 0x00000001069eb376 liblua_plugin.dylib`run(p_this=0x0000000100339020, psz_filename="/sarnobat.garagebandbroken/trash/vlc/share/lua/meta/reader/filename.lua", L=0x00000001003392f0, luafunction="read_meta", p_context=0x0000000000000000) at meta.c:128
    frame #9: 0x00000001069eaa79 liblua_plugin.dylib`read_meta(p_this=0x0000000100339020, psz_filename="/sarnobat.garagebandbroken/trash/vlc/share/lua/meta/reader/filename.lua", p_context=0x0000000000000000) at meta.c:213
    frame #10: 0x00000001069f069a liblua_plugin.dylib`vlclua_scripts_batch_execute(p_this=0x0000000100339020, luadirname="meta/reader", func=(liblua_plugin.dylib`read_meta at meta.c:204), user_data=0x0000000000000000) at vlc.c:281
    frame #11: 0x00000001069ea9c9 liblua_plugin.dylib`ReadMeta(p_this=0x0000000100339020) at meta.c:247
    frame #12: 0x0000000100107e66 libvlccore.dylib`generic_start(func=0x00000001069ea980, ap=0x00007ffeefbfe0a0) at modules.c:356
    frame #13: 0x0000000100107b83 libvlccore.dylib`module_load(obj=0x0000000100339020, m=0x00000001003264c0, init=(libvlccore.dylib`generic_start at modules.c:352), args=0x00007ffeefbfe390) at modules.c:183
    frame #14: 0x00000001001076a2 libvlccore.dylib`vlc_module_load(obj=0x0000000100339020, capability="meta reader", name="", strict=false, probe=(libvlccore.dylib`generic_start at modules.c:352)) at modules.c:279
    frame #15: 0x0000000100107de1 libvlccore.dylib`module_need(obj=0x0000000100339020, cap="meta reader", name=0x0000000000000000, strict=false) at modules.c:371
    frame #16: 0x0000000100150de9 libvlccore.dylib`InputSourceMeta(p_input=0x0000000101006ef0, p_source=0x0000000100453350, p_meta=0x0000000100338e40) at input.c:2825
    frame #17: 0x0000000100146e67 libvlccore.dylib`Init(p_input=0x0000000101006ef0) at input.c:1388
    frame #18: 0x0000000100146a58 libvlccore.dylib`input_Read(p_parent=0x0000000100337f80, p_item=0x0000000100337c60) at input.c:150
    frame #19: 0x0000000100119c0b libvlccore.dylib`playlist_MLLoad(p_playlist=0x0000000100335b10) at loadsave.c:169
    frame #20: 0x0000000100115b9c libvlccore.dylib`playlist_Create(p_parent=0x0000000100415880) at engine.c:257
    frame #21: 0x0000000100111ef3 libvlccore.dylib`intf_GetPlaylist(libvlc=0x0000000100415880) at interface.c:158
    frame #22: 0x0000000100112079 libvlccore.dylib`libvlc_InternalAddIntf(libvlc=0x0000000100415880, name="http,none") at interface.c:216
    frame #23: 0x00000001000df45f libvlccore.dylib`libvlc_InternalInit(p_libvlc=0x0000000100415880, i_argc=5, ppsz_argv=0x00007ffeefbfe990) at libvlc.c:333
    frame #24: 0x00000001000a747c libvlc.dylib`libvlc_new(argc=4, argv=0x00007ffeefbfea30) at core.c:60
    frame #25: 0x000000010000341e vlc-osx-static`main(i_argc=2, ppsz_argv=0x00007ffeefbfec28) at darwinvlc.m:281
    frame #26: 0x0000000100002ca4 vlc-osx-static`start + 52
```
 

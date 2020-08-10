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

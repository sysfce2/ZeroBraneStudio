---
layout: default
title: Frequently Asked Questions
---

<ul id='toc'>&nbsp;</ul>

## What does the bluish line under or around some names mean?

It identifies **function calls**; thit indicator is disabled by default starting from v0.70.
Depending on your current style it may also be shown as a solid line or a rounded box around a function name.
You can [enable/disable it](https://github.com/pkulchenko/ZeroBraneStudio/blob/master/cfg/user-sample.lua#L98),
[change its type](https://github.com/pkulchenko/ZeroBraneStudio/blob/master/cfg/user-sample.lua#L104),
or [change its color](https://github.com/pkulchenko/ZeroBraneStudio/blob/master/cfg/user-sample.lua#L101).
(Note that before v0.38 you'd need to set/modify `styles.fncall` instead of `styles.indicator.fncall`.)

## What are the dotted and dashed lines under variable names?

These are **scope indicators**; you can [change their types, colors, or disable them](doc-styles-color-schemes#indicators).

## How to specify a directory my script should be executed in for `require` commands to work?

You can set the [project directory](doc-getting-started#project-directory), which will be used to set the current directory when your application is run or debugged.

## Why stepping into function calls doesn't work in some cases?

You need to have a file opened in the IDE before you can step into functions defined in that file.
You can also configure the IDE to [auto-open files](https://github.com/pkulchenko/ZeroBraneStudio/blob/master/cfg/user-sample.lua#L71) for you.

## Why do I get a warning about attempt to connect to port 8172 when I start debugging?

The IDE is using port 8172 to communicate with the application being debugged.
If you get a firewall warning, you need to **allow the connection** for the debugging to work properly.

## Why am I getting "could not connect to ...:8172" message?

This may happen for several reasons:

- You start your application that uses `require('mobdebug').start()` call to connect to the IDE, but the **debugger server is not started** in the IDE.
You can fix this by selecting `Project | Start Debugger Server`; if it is disabled, the server is already started.
- Your **firewall is configured to block connection** to port 8172, which is used by the IDE to communicate with the application you are debugging.
You need to allow this connection for the debugging to work.
- In rare cases the IDE may **incorrectly detect the hostname** of the computer it runs on, which may prevent the debugging from working.
The **hostname is shown** in the Output window when the debugging is started: `Debugger server started at <hostname>:<port>`.
You can use a different hostname by setting `debugger.hostname` value in the [configuration file](doc-general-preferences#debugger).
For example, if the default hostname is incorrect, **try setting it to `localhost`** by using `debugger.hostname = "localhost"`.
- You may be **on VPN**, which may **block connections** or cause the IDE to **incorrectly detect the hostname**.
You may configure `debugger.hostname` as described above to see if this resolves the issue.

## Why breakpoints are not triggered?

Breakpoints set in source files may not work for several reasons:

- A breakpoint may be **inside a coroutine**; by default breakpoints inside coroutines are not triggered.
To enable debugging in coroutines, including triggering of breakpoints, you may either
(1) add `require('mobdebug').on()` call to that coroutine, which will enable debugging for that particular coroutine, or
(2) add `require('mobdebug').coro()` call to your script, which will enable debugging for all coroutines created using `coroutine.create` later in the script.
- If you enable coroutine debugging using `require('mobdebug').coro()`, this will **not affect coroutines created using C API** or Lua code wrapped into `coroutine.wrap`.
You can still debug those fragments after adding `require('mobdebug').on()` to the coroutine code. 
- The path of the file known to the IDE **may not be the same** as the path known to the Lua engine.
For example, if you use an embedded engine, you may want to check if the path reported by the engine is normalized (doesn't include `../` references) by checking the result of `debug.getinfo(1,"S").source`.
- The script you are debugging may **change the current folder** (for example, using `lfs` module) and load the script (using `dofile`) from the changed folder.
To make breakpoints work in this case you may want to **use absolute path** with `dofile`.
- You may be using your own Lua engine that doesn't report file names relative to the project directory (as set in the IDE).
For example, you set the project directory pointing to `scripts` folder (with `common` subfolder) and the engine reports the file name as `myfile.lua` instead of `common/myfile.lua`;
the IDE will be looking for `scripts/myfile.lua` instead of `scripts/common/myfile.lua` and the file will not be activated and the breakpoints won't work.
You may also be using inconsistent path separators in the file names; for example, `common/myfile.lua` in one case and `common\myfile.lua` in another.
- If you are loading files using `luaL_loadbuffer`, make sure that the chunk name specified (the last parameter) matches the file location.

## Is it possible to debug dynamic fragments loaded with `loadstring()`?

Yes; starting from v0.38 if you step into `loadstring()` call, the IDE will open a new window (when [editor.autoactivate](https://github.com/pkulchenko/ZeroBraneStudio/blob/master/cfg/user-sample.lua#L71) is enabled) with the code you can then step through.
As an alternative, you can specify a filename as the second parameter to `loadstring` (also as the absolute or relative path) and have that file in your project directory with the same content as what's loaded with `loadstring`.

You can then open that file in the IDE or configure it to [auto-open](https://github.com/pkulchenko/ZeroBraneStudio/blob/master/cfg/user-sample.lua#L71) it for you.

## Is debugging of Lua 5.2 applications supported?

Yes; see [Lua 5.2 debugging](doc-lua52-debugging) section for details.

## Is debugging of Lua 5.3 applications supported?

Yes; see [Lua 5.3 debugging](doc-lua53-debugging) section for details.

## Is debugging of LuaJIT applications supported?

Starting from v0.35 the debugging of LuaJIT applications is supported out-of-the-box.
See [LuaJIT debugging](doc-luajit-debugging) section for details.

## Why am I getting compilation errors in the IDE when my code runs fine outside of it?

Starting from v0.39, ZeroBrane Studio is using LuaJIT as its internal Lua engine.
LuaJIT is a bit more strict than Lua 5.1 in some checks and may return errors even when your application runs fine by Lua 5.1.
One typical example is string escape sequences. Lua 5.1 lists [escape sequences it recognizes](http://www.lua.org/pil/2.4.html), but it will accept other sequences as well, for example, `\/`.
Running a script with this escape sequence under LuaJIT will trigger an error: `invalid escape sequence near ...`.

The solution in this case is to **"fix" the escape sequence** and replace `\/` with `/`, which will have the same effect in Lua 5.1, LuaJIT, and Lua 5.2.

## Why is the text blurry when running on Windows 8?

Right-click on ZeroBrane Studio icon -> `Properties` -> `Compatibility` -> `"Disable display scaling on high DPI settings"`.
See the link in [this ticket](https://github.com/pkulchenko/ZeroBraneStudio/issues/210) for alternative solutions if this doesn't work.

## Why is the text blurry in the editor when running on retina display (OSX)?

You can set `hidpi = true` in [configuration settings](doc-configuration);
this setting is enabled by default on OSX starting from v0.60.

Note that using this setting negatively affects [indicators](doc-styles-color-schemes#indicators) that have alpha property (the indicators are not shown when this setting is enabled).

## How to change background color in the editor?

You can specify `styles.text.bg = {240,240,240}` in [configuration settings](doc-configuration).
See the [example](https://github.com/pkulchenko/ZeroBraneStudio/blob/master/cfg/user-sample.lua).
To modify colors and appearance of IDE elements, check [documentation on styles and color schemes](doc-styles-color-schemes).

## How to change the color scheme in the editor?

You can open `cfg/scheme-picker.lua` in the IDE and click on links with color scheme names to test included schemes.
These schemes can be configured as shown in [user-sample.lua](https://github.com/pkulchenko/ZeroBraneStudio/blob/master/cfg/user-sample.lua#L83-L88).

For more information on how to modify colors and appearance of IDE elements, check [documentation on styles and color schemes](doc-styles-color-schemes).

## How to accept keyboard input for applications started from the IDE?

You need to output something using `print` or `io.write` before reading input.
You will see a prompt in the Output window where you can enter your input.

If you launch the application outside of the IDE, make sure you **flush the printed output** (or use `io.stdout:setvbuf('no')`) before accepting the input.

## Why don't I see `print` output immediately in the Output window?

Lua (and some engines based on it, like Love2d) has output buffered by default,
so if you only `print` a small number of bytes, you may see the results only after the script is completed.

If you want to see the `print` output immediately, **add `io.stdout:setvbuf("no")` to your script**, which will turn the buffering on.
There may be a small performance penalty as the output will be flushed after each `print`, so keep that in mind.

## How to pass command line arguments?

This is described in the [command line parameters](doc-general-preferences#command-line-parameters) section.

## Where is the configuration file stored?

This is covered in the description of system [configuration](doc-configuration).

## How can I modify a key mapping?

To modify a key mapping for a particular menu item, see the [key mapping](doc-general-preferences#key-mapping) section.
You may also review [xcode-keys](https://github.com/pkulchenko/ZeroBraneStudio/blob/master/cfg/xcode-keys.lua) configuration file that can be used to modify keyboard shortcuts to match those in XCode.

## Why am I getting a warning about "invalid UTF8 character" in the files I open in the editor?

You probably have files created using one of Microsoft [Windows code pages](http://en.wikipedia.org/wiki/Windows_code_page#List); most likely 1252, 936, or 950.
ZeroBrane Studio is using [UTF-8 encoding](http://en.wikipedia.org/wiki/UTF-8), which allows users to use any supported language, but the files need to use Unicode encodings.
If you want to continue editing this file in the IDE, you have two choices:

1. Ignore the warning and **change `[SYN]` characters** to whatever text you need *inside ZeroBrane Studio*.
2. **Convert your files** from the encoding you are using to UTF-8 (65001) and then load them into the IDE.

## How do I run multiple instances of ZeroBrane Studio?

You can start multiple instances if you disable "single instance" check
either by adding `singleinstance=false` to the configuration file
or by starting the second instance with the following command: `zbstudio -cfg "singleinstance=false"`.

The single instance check is disabled on OSX by default.

## How do I start two ZeroBrane Studio instances to debug two applications at the same time?

You can start one instance as you normally do and then start the second instance using the following command: `zbstudio -cfg "singleinstance=false; debugger.port = 8173"`.
This command disables a singleinstance check for the second instance and configures it to use port 8173 for debugging.
You can then use `require('mobdebug').start("domain-name", 8173)` in your application to connect to the second instance for debugging.

If you use Mac OSX, you may need to run the command as `open ZeroBraneStudio.app --args -cfg "singleinstance=false; debugger.port = 8173"`.

## How do I restore default configuration for recent files, projects, and editor tabs?

You can **remove** the [configuration file](doc-general-preferences#session-configuration) ZeroBrane Studio is using to store these settings.

## Why can't I delete text that includes formatted comments?

Formatted comments allow usage of [Markdown formatting](doc-markdown-formatting), which uses styles with hidden characters.
Those characters can't be deleted with some of the delete operations (line 'cut' or 'delete selection') and need to be deleted using `Delete` or `Backspace`.

## I'm getting a message about mixed end-of-line sequences. How do I make them visible?

You may run `GetEditor():SetViewEOL(1)` in the local console; you will see end-of-line sequences marked as `CR`, `LF`, or `CRLF`.
Re-open the document or run `GetEditor():SetViewEOL(0)` to disable showing end-of-line sequences.

## How to make the current line more visible during debugging?

You can try changing the current line marker to something more visible, like underline by adding the following code to the configuration file: `styles.marker.currentline.ch = wxstc.wxSTC_MARK_UNDERLINE`.
You can also change the color and the type of the marker: `styles.marker.currentline = {ch = wxstc.wxSTC_MARK_UNDERLINE, bg = {0, 0, 255}}` (this will set the background to `blue`).

## Where do I find the list of functions in the current file?

Before v0.90, the list of functions was available through the dropdown in the toolbar.
Starting from v0.90, the list of functions is available through a separate tab located next to the project tab.
The content of the Outline can be [configured in various ways](doc-general-preferences#outline).

## Why am I getting `mach-o, but wrong architecture` error on OSX?

You may be trying to load a 64bit module using Lua executable that is included with the IDE.
Since the Lua executable is 32bit, you get the error about incompatible architectures.

To avoid this issue you need to configure the IDE to use your own 64bit Lua executable by setting `path.lua = '/full/path/lua-executable'` in the configuration.

Note that you can still use the IDE to debug your applications as the luasocket library required for debugging is compiled as universal binary and can be loaded into both 32bit and 64bit applications.

## How do I show Corona SDK Console window on Windows?

(0.81+) You may set `corona = {showconsole = true}` in [system or user configuration](doc-configuration). See [all Corona-specific preferences](doc-corona-preferences) for details.

(0.80 and earlier versions) You may add the following line to [system or user configuration](doc-configuration): `unhidewindow.ConsoleWindowClass = 0`.
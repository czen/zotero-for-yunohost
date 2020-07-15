# Zotero Word for Mac Integration

This is a Firefox add-on that consists of a library written in ObjC that communicates with Microsoft Word out of process using AppleScript, a js-ctypes wrapper for said library, and a template that is installed into Microsoft Word to communicate with Zotero.

## ObjC Library Build Requirements
- [MacOSX10.7.sdk](https://github.com/phracker/MacOSX-SDKs/tree/master/MacOSX10.7.sdk) symlinked to `/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs`
- XCode 8.2+

### To Build the ObjC Library
- Launch the project in `build/src/libZoteroMacWordIntegration.xcodeproj`
- Product -> Build For -> Profiling

## Template Build Requirements
- Templates should be built with the oldest version of Word to be supported. Otherwise older versions of Word may fail to function properly. This is currently:
  - Word 2016 (for the ribbonized dotm template)
  - Word 2011 (for the old dot template)

### To Modify/Build the Templates
- Open the template from inside Microsoft Word
- Go to View->Macros->View Macros (Ribbonized Word) or Tools->Macros->View Macros (Word 2016) and click "Edit" for one of the Zotero macros
- Edit/replace code as desired
- Go to Debug->Compile Project to ensure there are no code errors
- Run `build/template/unpack_templates.sh`

## Development Starter's Guide

Start by opening the dotm/dot template in Word. Word templates have support for custom macros and adding UI elements
to call the macros, which is how the extension is implemented on Word. 
RibbonUI can be edited by extracting the dotm file. To edit the .dot template UI Word for Windows 2003 is needed.
In VBA Macros code you will see that calls to Zotero are
executed by writing commands to a file named `.zoteroIntegrationPipe`. The command format is:

```.bash
echo "MacWord2016 <commandName>" > $PIPE
```

The pipe is created in [zoteroMacWordIntegration2016Pipe.js](https://github.com/zotero/zotero-word-for-mac-integration/blob/823fd6daed429b43070eab2ea2195384944d58d6/components/zoteroMacWordIntegration2016Pipe.js#L34-L34)
for Word 2016 or [integration.js](https://github.com/zotero/zotero/blob/aa783878dee10ebb9f0649593ac52354d51947c7/chrome/content/zotero/xpcom/integration.js#L68-L68)
for Word 2011 and older, and requests are handled in [integration.js](https://github.com/zotero/zotero/blob/aa783878dee10ebb9f0649593ac52354d51947c7/chrome/content/zotero/xpcom/integration.js#L2273-L2273).

Zotero talks to Word via [js-ctypes bindings](https://github.com/zotero/zotero-word-for-mac-integration/blob/00d9743f40649178b0eaa39d7d0ae2427e714bb1/components/zoteroMacWordIntegration.js#L74-L74)
to an [ObjC library](https://github.com/zotero/zotero-word-for-mac-integration/blob/f6d9042e35892050a717f7837e05b63f6bcb3135/build/src/zoteroMacWordIntegration.h).
The ObjC library itself utilises ScriptBridge to call [AppleScript commands](https://github.com/zotero/zotero-word-for-mac-integration/blob/952ac04bf2748a79c8cc54eadc1a8f8cabacbc2d/build/src/Word.h)
and interact with a running Word process. Header files, such as `Word.h`, are generated by running

```.bash
sdef /Applications/Microsoft\ Word.app | sdp -fh --basename Word
```

as explained in [Scripting Bridge Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ScriptingBridgeConcepts/UsingScriptingBridge/UsingScriptingBridge.html).
AppleScript command definitions can be found within ScriptEditor > File > Open Dictionary...
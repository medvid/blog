---
title: Microsoft Visual Studio configuration
date: 2015-02-02 00:00:00 Z
---

* [Disable file preview in VS2012](http://stackoverflow.com/questions/10952185/disable-file-preview-in-vs2012)

  In Tools > Options > Environment > Tabs and Windows, you can disable it by unckecking "Solution explorer" under Preview tab.

* [Display Line Numbers in the Editor](https://msdn.microsoft.com/en-us/library/ms165340.aspx)

  On the menu bar, choose Tools, Options. Expand the Text Editor node, and then select either the node for the language you are using, or All Languages to turn on line numbers in all languages. Or you can type line number in the Quick Launch box.
  
* [Turning off automatic outlining in Visual Studio](http://stackoverflow.com/questions/1709212/turning-off-automatic-outlining-in-visual-studio)

  Tools -> Options -> Text Editor -> C# -> Advanced -> Enter outlining mode when files open.

* [Fix WinForms Designer errors triggered by custom controls](https://msdn.microsoft.com/en-us/library/ms171843.aspx)

  Add to `c:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\IDE\devenv.exe.config`:
  
      <appSettings>
        <add key="EnableOptimizedDesignerReloading" value="false" />
      </appSettings>

* [Force syntax highlighting for large XML files](http://superuser.com/questions/509438/how-to-force-syntax-highlighting-for-large-xml-files)

    Set to 40 MB (default=10MB):

        HKCU\Software\Microsoft\VisualStudio\12.0_Config\XmlEditor\MaxFileSizeSupportedByLanguageService

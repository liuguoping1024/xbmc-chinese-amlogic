xbmc-amlogic-chinese, based on xbmc-15.0-Isengard

This is a branch release for amlogic.

## 1. Fixed the video size error bug.

The video size may be zoom in/zoom out sometimes on S802/S805/... CPU etc, because the official KODI/XBMC
does not list all the display mode for amlogic on the sys path"/sys/class/display/mode". So Kodi will make
mistakes when calculate dimension under the display modes.

 such as:

- 720p59hz(\n)
- 1080p59hz(\n)
- ...

You should use strncmp(), not StringUtil::CompareNoCase() ...

The modification is in the function:
`bool aml_mode_to_resolution(const char *mode, RESOLUTION_INFO *res);`

Here:
`
  else if(strncmp(fromMode.c_str(), "720p59hz", 8) == 0)
  //else if (StringUtils::EqualsNoCase(fromMode, "720p59hz"))
  {
    res->iWidth = 1280;
    res->iHeight= 720;
    res->iScreenWidth = 1280;
    res->iScreenHeight= 720;
    res->fRefreshRate = 60;
    res->dwFlags = D3DPRESENTFLAG_PROGRESSIVE;
  }

`

And Here:
`
  else if(strncmp(fromMode.c_str(), "1080p59hz", 9) == 0)
  //else if (StringUtils::EqualsNoCase(fromMode, "1080p59hz"))
  {
    res->iWidth = 1920;
    res->iHeight= 1080;
    res->iScreenWidth = 1920;
    res->iScreenHeight= 1080;
    res->fRefreshRate = 60;
    res->dwFlags = D3DPRESENTFLAG_PROGRESSIVE;
  }

`
And, sometimes, the video display rectangle is wrong... just refresh it ...(v15.0 is error, while 14.2 is ok).

AMLCodec.cpp: line 1688:

`
  if (strScaler.find("enabled") == std::string::npos)     // Scaler not enabled, use screen size
  {                                                                                                                                               
     //m_display_rect = CRect(0, 0, CDisplaySettings::Get().GetCurrentResolutionInfo().iScreenWidth, CDisplaySettings::Get().GetCurrentResolutionInfo().iScreenHeight);
    std::string mode;
    SysfsUtils::GetString("/sys/class/display/mode", mode);

    RESOLUTION_INFO res;
    if (aml_mode_to_resolution(mode.c_str(), &res))
      m_display_rect = CRect(0, 0, res.iScreenWidth, res.iScreenHeight);
    else
      m_display_rect = CRect(0, 0, CDisplaySettings::Get().GetCurrentResolutionInfo().iScreenWidth, CDisplaySettings::Get().GetCurrentResolutionInfo().iScreenHeight);
  }
`
Normally, if you do not want to changed the source code, just set the display mode to 1080p is a good suggestion.

## 2. Remove other languages, ~~~ Chinese/English only! 

This version is designed for Chinese User.So. only Simplify Chinese remained. 

## 3. Remove extra addon, such as PVR.

Did you find that the PVR is always disabled on Android Platform?

I did. 

So all the pvr related plugin is dropped.

## 4. see also : http://www.hdpfans.com/thread-526265-1-1.html

## 5. Remove "Waiting for external storage"
A delayed runnable can pass the storage check, but it will not always succed.


Enjoy yourself!

All the apk is stored here:
http://pan.baidu.com/s/1jGpKmJc



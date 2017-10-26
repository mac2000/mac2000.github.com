---
layout: post
title: AutoHotkey window snapper tool

tags: [active, autohotkey, bottom, delete, end, home, insert, left, MinMax, MonitorWorkArea, pgdown, pgup, right, snap, SysGet, top, window, WinGet, WinGetPos, WinMaximize, WinMove, WinRestore, WS_SIZEBOX]
---

How it works:

http://www.youtube.com/watch?v=pvD6S7mqsR4

This tool will move and resize your active window in next way:

![screenshot](https://github.com/mac2000/WinSnapper/raw/master/WinSnapper.png)

Something like:

![screenshot](https://github.com/mac2000/WinSnapper/raw/master/Screen.png)

WinSnapper [sources](https://github.com/mac2000/WinSnapper/blob/master/WinSnapper.ahk) and [executable](https://github.com/mac2000/WinSnapper/raw/master/WinSnapper.exe)

And here is code:

```
; WinSnapper:
; Move and resize windows with arrow and text navigation keys
; Screen is splitted on 2 / 3 + 1 / 3 sides, 1024 is preffered left side width,
; so u can snap your browser to left side and still see whole page
;
; Insert, Delete, Home, End, Page Up, Page Down - snaps active window to desired side of screen
; Arrow keys works almost same way as Win + Left(Right)
;
; Alternatives:
; http://www.autohotkey.com/community/viewtopic.php?t=21703&postdays=0&postorder=asc&start=0
; http://www.autohotkey.com/community/viewtopic.php?t=12229
; http://www.autohotkey.com/community/viewtopic.php?t=53674

; Some global variables

mw := GetMonitorWidth()             ; Monitor width
mh := GetMonitorHeight()            ; Monitor height
pw := mw / 3                        ; 1 / 3 of monitor width
ph := mh / 3                        ; 1 / 3 of monitor height
lw := 2 * pw > 1024 ? 1024: 2 * pw  ; Left side width (1024 is preffered)
rw := mw - lw                       ; Right side width
th := round(2 * ph)                 ; Top side height
bh := ceil(ph)                      ; Bottom side height

^#Insert::MoveWindowLeftTop()
^#Delete::MoveWindowLeftBottom()
^#Home::MoveWindowTop()
^#End::MoveWindowBottom()
^#PgUp::MoveWindowRightTop()
^#PgDn::MoveWindowRightBottom()

^#Up::MaximizeToggle() ;WinMaximize A
^#Down::MoveWidnowCenter()
^#Left::MoveWindowLeft()
^#Right::MoveWindowRight()

MoveWindowLeftTop() {
    global
    if (IsResizable()) {
        WinMove, A,, 0, 0, lw, th
    } else {
        WinMove, A,, 0, 0
    }
}

MoveWindowLeftBottom() {
    global
    if (IsResizable()) {
        WinMove, A,, 0, th, lw, bh
    } else {
        WinMove, A,, 0, th
    }
}

MoveWindowRightTop() {
    global
    if (IsResizable()) {
        WinMove, A,, lw, 0, rw, th
    } else {
        WinMove, A,, lw, 0
    }
}

MoveWindowRightBottom() {
    global
    if (IsResizable()) {
        WinMove, A,, lw, th, rw, bh
    } else {
        WinMove, A,, lw, th
    }
}

MoveWindowLeft() {
    global
    if (IsResizable()) {
        WinMove, A,, 0, 0, lw, mh
    } else {
        WinMove, A,, 0, 0
    }
}

MoveWindowRight() {
    global
    if (IsResizable()) {
        WinMove, A,, lw, 0, rw, mh
    } else {
        WinMove, A,, lw, 0
    }
}

MoveWindowTop() {
    global
    if (IsResizable()) {
        WinMove, A,, 0, 0, mw, th
    } else {
        WinMove, A,, 0, 0
    }
}

MoveWindowBottom() {
    global
    if (IsResizable()) {
        WinMove, A,, 0, th, mw, bh
    } else {
        WinMove, A,, 0, th
    }
}

MoveWidnowCenter() {
    global
    if (IsResizable()) {
        WinMove, A,, round(mw / 2 - lw / 2), round(mh / 2 - th / 2), lw, th
    } else {
        WinGetPos,,, Width, Height
        WinMove, A,, round(mw / 2 - Width), round(mh / 2 - Height)
    }
}

; Helper functions
GetMonitorWidth() {
    SysGet, mon, MonitorWorkArea
    return monRight - monLeft
}

GetMonitorHeight() {
    SysGet, mon, MonitorWorkArea
    return monBottom - monTop
}

IsResizable() {
    WinGet, Style, Style, A
    return (Style & 0x40000) ; WS_SIZEBOX
}

MaximizeToggle() {
    WinGet, state, MinMax, A
    if (state && IsResizable()) {
        WinRestore, A
    } else {
        WinMaximize, A
    }
}
```

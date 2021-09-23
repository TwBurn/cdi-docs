# C-BINDING MPEG VIDEO
> This document is a reconstruction as any original documentation seems to be lost. Notes by the creator of this document are listed in quote sections like this one.
>
> Please refer to the Green Book description of the corresponding `MV_` function for complete information on function parameters and behaviour..
>
> Any original document was probably

(C) Copyright 1993, Philips Consumer Electronics B.V.

## Table of contents
- [`mv_abort()`](#mv_abort-abort-the-currently-executed-play) Abort the currently executed play
- [`mv_borcol()`](#mv_borcol-xxx) XXX
- [`mv_chspeed()`](#mv_chspeed-xxx) XXX
- [`mv_close()`](#mv_close-free-the-used-audio-descriptor) Free the used video descriptor
- [`mv_cncl()`](#mv_cncl-xxx) XXX
- [`mv_continue()`](#mv_continue-continue-a-paused-play) Continue a paused play
- [`mv_create()`](#mv_create-reserve-and-initialize-a-videoo-map-descriptor) Reserve and initialize a video map descriptor
- [`mv_freeze()`](#mv_freeze-xxx) XXX
- [`mv_hide()`](#mv_hide-xxx) XXX
- [`mv_imgsize()`](#mv_imgsize-xxx) XXX
- [`mv_info()`](#mv_info-return-a-pointer-to-the-video-map-descriptor) Return a pointer to the video map descriptor
- [`mv_jump()`](#mv_jump-compensate-video-streams-timing-parameters-on-behalf-of-seamless-jump) Compensate video streams timing parameters on behalf of seamless jump
- [`mv_loop()`](#mv_loop-xxx) XXX
- [`mv_cdnext()`](#mv_cdnext-xxx) XXX
- [`mv_hostnext()`](#mv_hostnext-xxx) XXX
- [`mv_off()`](#mv_off-xxx) XXX
- [`mv_org()`](#mv_org-xxx) XXX
- [`mv_pause()`](#mv_pause-pause-the-current-asynchronous-play) Pause the current asynchronous play
- [`mv_cdplay()`](#mv_cdplay-play-video-from-cd) Play video from CD
- [`mv_hostplay()`](#mv_hostplay-start-play-of-a-host-video-map) Start play of a host video map
- [`mv_pos()`](#mv_pos-xxx) XXX
- [`mv_release()`](#mv_release-xxx) XXX
- [`mv_selstrm()`](#mv_selstrm-xxx) XXX
- [`mv_show()`](#mv_show-xxx) XXX
- [`mv_slink()`](#mv_slink-link-to-external-subroutine-module)-Link to external subroutine module
- [`mv_status()`](#mv_status-xxx) XXX
- [`mv_trigger()`](#mv_trigger-define-events-to-signal) Define events to signal
- [`mv_window()`](#mv_window-xxx) XXX

## INTRODUCTION
The functions described in this chapter are additions to the standard OS-9 library.

The library has the following name:
- `mvcdi.l` c-bindings with parameter checking
- `mvcdin.l` c-bindings without parameter checking

A C-program accessing the in this chapter mentioned functions must
link with the following library files. For example:

```
        cc prog.r -l=/dd/lib/mvcdi.l
                  -l=/dd/lib/fmv.l -f=prog
```

All constants and structure definitions are made in DEFS file `mv.h`.

### `mv_abort()` Abort the currently executed play

```
mv_abort(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

This function aborts the play that is currently being executed.
The play will not be active any longer and
fields in the map descriptor are reset. XXX The last
video picture displayed will remain visible. This is a privileged call
and may used only by a process with a userID of the
super user or with the userID of the process which
started the operation. If no play is active, an abort
error will be returned.

If this play was running in synchronized mode with
an audio play, then this call will end the synchronized
mode. The audio playback will continue in non-synchronized
mode.

`mv_abort()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_borcol()` XXX

```
mv_borcol(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_borcol()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_chspeed()` XXX

```
mv_chspeed(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_chspeed()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_close()` Free the used video descriptor

```
mv_close(path,vimid)
int     path,     /*path number*/
        vimid;    /*video mapID*/
```

This function aborts any ongoing actions on the
specified video mapID and frees the used descriptor.
This is a privileged call and may be used only by a
process with a userID of the super user or with the
userID of the process which created the video mapID.

`mv_close()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_cncl()` XXX

```
mv_cncl(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_cncl()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_continue()` Continue a paused play

```
mv_continue(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_continue()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_create()` Reserve and initialize a video map descriptor

```
mv_create(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_create()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_freeze()` XXX

```
mv_freeze(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_freeze()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_hide()` XXX

```
mv_hide(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_hide()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_cdnext()` XXX

```
mv_cdnext(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_cdnext()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_hostnext()` XXX

```
mv_hostnext(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_hostnext()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_imgsize()` XXX

```
mv_imgsize(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_imgsize()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_info()` Return a pointer to the video map descriptor

```
MVmapDesc *mv_info(path,vimid)
int     path,    /*path number*/
        vimid;   /*video mapID*/
```

This function returns a pointer to the video map
descriptor corresponding to the given video mapID.
The fields in the descriptor are for information
purposes only. Fields should only be changed by
calling the appropriate functions.
The video map descriptor has the following format:
```
typedef struct mvmapdesc {
   XXX
} MVmapDESC;
```

`mv_info()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_jump()` Compensate video streams timing parameters on behalf of seamless jump

```
mv_jump(path,delta)
int     path,     /*path number*/
        delta;    /*delta value*/
```

> In earlier DVC ROMs this function was notoriously unreliable, so much so that
> no released CD-i titles actually use this function.

The application is allowed, when playing from CD, to
discontinue the current data stream and to start
supplying data that comes from another position. To
overcome a discontinuity in timing parameters, this
function provides the application a means to adapt the
new timing parameters in such a way that they fit
seamless into the old stream. As soon as either a play
is started or the decoder detects a discontinuity in
the supplied stream, it will start correcting the
timing parameters with a given delta value. This delta
value is defined as SCR(j) - SCR(i) and is given in a
resolution of 22. 5 kHz. SCR( j) is the value of the
first sector of the new data and SCR(i) is the value
of the SCR of the sector that would have been supplied
next if no jump was done. Initially the delta value is
zero. After a play is aborted, the delta value will
be reset to zero. When playing from host, the delta
value will have no influence at all.

Since a discontinuity in SCR is detected only if the
difference in between SCRs is negative or greater than
0. 70 seconds, a positive delta value less than ore
equal to 15,750 is considered as an illegal parameter.

`mv_jump()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_loop()` XXX

```
mv_loop(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_loop()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_off()` XXX

```
mv_off(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_off()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_org()` XXX

```
mv_org(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_org()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_pause()` Pause the current asynchronous play

```
mv_pause(path)
int     path;  /*path number*/
```

This function pauses the asynchronous play that is
currently being executed. This is a privileged
function call and may be used only be a process with
a userID of the super user or with the userID of the
process that started, the play operation. If no
asynchronous play is active at the time this
function is called, or if the play is already
paused, a bad mode error will be returned.

If the video play is in synchronized mode with
audio, this function will pause both the audio and
the video playback.

`mv_pause()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_cdplay()` Play video from CD

```
mv_cdplay(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_cdplay()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_hostplay()` Start play of a host video map

```
mv_hostplay(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_hostplay()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_pos()` XXX

```
mv_pos(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_pos()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_release()` XXX

```
mv_release(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_release()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_selstrm()` XXX

```
mv_selstrm(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_selstrm()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_show()` XXX

```
mv_show(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_show()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_slink()` Link to external subroutine module

```
mv_slink(path,subptr)
int  path,         /*path number*/
     subptr;       /*ptr. name of subr. module*/
```

This function is used as a generalized method of
expanding the command set of MoviMan. It utilizes
a special subroutine module which is linked into
MoviMan. When MoviMan receives an unknown GetStat
or Setstat service request, it will pass the call
to the subroutine module. If it is unknown to the
subroutine module, the subroutine module should
then pass the call to the driver.
There are four significant entry points into this
external subroutine module: `Init`, `SetStat`, `GetStat`
and `Close`.

`Init` will be called when the module is first
linked to by MoviMan. It is provided to allow the
module to initialize data structures and
variables. Likewise, the `Close entry` point will be
called when the last path to MoviMan is closed. At
this time the subroutine module should free any
memory which was allocated during its execution.

The `SetStat` and `GetStat` entry points are provided
to actually do the main work of the subroutine
module.

It is impossible for two different subroutine
modules to be linked at the same time. This
applies even when they are linked by different
processes. However, more than one process may be
linked to the same subroutine module.

`mv_slink()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_status()` XXX

```
mv_status(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_status()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_trigger()` Define events to signal

```
mv_trigger(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_trigger()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.

### `mv_window()` XXX

```
mv_windows(path,xxx)
int     path,  /*path number*/
        xxx;   /*xxx*/
```

XXX

`mv_windows()` function returns 0 if successful,
otherwise -1 is returned and `errno` is set.



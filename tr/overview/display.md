```eval_rst
:github_url: https://github.com/littlevgl/docs/blob/master/tr/overview/display.md
```
# Displays

``` important:: The basic concept of *Display* in LittlevGL is explained in the [Porting](/porting/display) section. So before reading further, please read that section first.
```
In LittlevGL you can have multiple displays each with its own drivers and objects. 

Creating more displays is easy: just initialize display buffers and register the drivers for every display. 
When you create the UI use `lv_disp_set_deafult(disp)` to tell the library to which display create the object. 


But in which cases can you use the multi-display support? Here are some examples:
- Have a "normal" TFT display with local UI and create "virtual" screens on VNC on demand. (You need to add your own VNC driver)
- Have a large TFT display and a small monochrome display.
- Have some smaller and simple displays in a large instrument or technology
- Have two large TFT displays: one for a customer and one for the shop assistant

### Using only one display
Using more displays can be useful but in most of the cases, it's not required. Therefore the whole concept of multi-displays is completely hidden if you register only one display.
By default, the lastly created (the only one) display is used as default. 

`lv_scr_act()`, `lv_scr_load(scr)`, `lv_layer_top()`, `lv_layer_sys()`, `LV_HOR_RES` and `LV_VER_RES` are always applied on the lastly created (default) screen.
If you pass `NULL` as `disp` parameter to display related function usually the default display will be used. 
E.g. `lv_disp_trig_activity(NULL)` will trigger a user activity on the default screen. (See below in [Inactivity](#Inactivity)).

### Mirror display

To mirror the image of display to an other display you don't really need to use the multi-display support. Just transfer the buffer received in `drv.flush_cb` to an other display too.

### Split image
You can create a larger display from more smaller ones. You do it like this:
1. Set the resolution of the displays to the large display's resolution 
2. In `drv.flush_cb` truncate and modify the `area` parameter for each display.
3. Send the buffer's content to each display with the truncated area, 

## Screens

Every display has it each set of [Screens](overview/object#screen-the-most-basic-parent) and the object on the screens. 

Screens can be considered the highest level containers which have no parent. 
The screen's size is always equal to its display's and size their position is (0;0). Therefore the screens coordinates can't be changed, i.e. `lv_obj_set_pos()`, `lv_obj_set_size()` or similar functions can't be used on screens.

A screen can be created from any object type but two most typical types are the [Base object](/object-types/obj) and the [Image](/object-types/img) (to create a wallpaper).

To create a screen use `lv_obj_t * scr = lv_<type>_create(NULL, copy)`. `copy` can be an other screen to copy it. 

To load a screen use `lv_scr_load(scr)`. The get active screen use `lv_scr_act()`. These functions works on the default display to specify which display you mean use `lv_disp_get_scr_act(disp)` and `lv_disp_load_scr(disp, scr)`.
 
Screens can be deleted with `lv_obj_del(scr)` but be sure to not delete currently loaded screen. 


### Opaque screen
Usually, the opacity of the screen is `LV_OPA_COVER` to provide a solid, folly covering background for its children. 
However, in some special case, you might want a transparent screen. For example, if you have a video player which renders the video frames on a layer but on an other layer you want to create an OSD menu (over the video) using LittlevGL. 
In this the style of the screen you should have `body.opa = LV_OPA_TRANSP` or `image.opa = LV_OPA_TRANSP` (or other `LV_OPA_...` values) to make the screen opaque. 
To properly handle the screens opacity `LV_COLOR_SCREEN_TRANSP` needs to be enabled. Not that, it works on with `LV_COLOR_DEPTH = 32`. 
The Alpha channel of 32-bit colors will be 0 where there are no objects and will be 255 where there are solid objects. 


## Features of displays

### Inactivity

The user's inactivity is measured on each display. Every use of an [Input device](/overview/indev) (if [associated with the display](/porting/indev#other-features)) counts as an activity. 
To get time elapsed since the last activity use `lv_disp_get_inactive_time(disp)`. If `NULL` is passed the overall smallest inactivity time will be returned from all displays. 

You can manually trigger an activity using `lv_disp_trig_activity(disp)`. If `disp` is `NULL` the default screen will be used. 


## Colors

The color module handles all color-related functions like changing color depth, creating colors from hex code, converting between color depths, mixing colors etc. 

The following variable types are defined by the color module:

- **lv_color1_t** Store monochrome color. For compatibility it also has R,G,B fields but they are always the same (1 byte)
- **lv_color8_t** A structure to store R (3 bit),G (3 bit),B (2 bit) components for 8 bit colors (1 byte)
- **lv_color16_t** A structure to store R (5 bit),G (6 bit),B (5 bit) components for 16 bit colors (2 byte)
- **lv_color32_t** A structure to store R (8 bit),G (8 bit), B (8 bit) components for 24 bit colors (4 byte)
- **lv_color_t** Equal to `lv_color1/8/16/24_t` according to color depth settings
- **lv_color_int_t** `uint8_t`, `uint16_t` or `uint32_t` according to color depth setting. Used to build color arrays from plain numbers.
- **lv_opa_t** A simple `uint8_`t type to describe opacity.

The `lv_color_t`, `lv_color1_t`, `lv_color8_t`, `lv_color16_t` and `lv_color32_t` types have got four fields:

- **ch.red** red channel
- **ch.green** green channel
- **ch.blue** blue channel
- **full** red + green + blue as one number

You can set the current color depth in *lv_conf.h* by setting the `LV_COLOR_DEPTH` define to 1 (monochrome), 8, 16 or 32.

### Convert color
You can convert a color from the current color depth to an other. The converter functions return with a number so you have to use the `full` field:

```c
lv_color_t c;
c.red   = 0x38;
c.green = 0x70;
c.blue  = 0xCC;

lv_color1_t c1;
c1.full = lv_color_to1(c);	/*Return 1 for light colors, 0 for dark colors*/

lv_color8_t c8;
c8.full = lv_color_to8(c);	/*Give a 8 bit number with the converted color*/ 

lv_color16_t c16;
c16.full = lv_color_to16(c); /*Give a 16 bit number with the converted color*/

lv_color32_t c24;
c32.full = lv_color_to32(c);	/*Give a 32 bit number with the converted color*/
```

### Swap 16 colors
You may set `LV_COLOR_16_SWAP` in *lv_conf.h* to swap the bytes of *RGB565* colors. It's useful if you send the 16 bit colors via a byte-oriented interface like SPI. 
As 16 bit numbers are stored in Little Endian format (lower byte on the lower address) the interface will send the lower byte first. However, displays usually need the higher byte first. A mismatch in the byte order will result in highly distorted colors.


### Create and mix colors
You can create colors with the current color depth using the LV_COLOR_MAKE macro. It takes 3 arguments (red, green, blue) as 8 bit numbers. 
For example to create light red color: `my_color = COLOR_MAKE(0xFF,0x80,0x80)`. 

Colors can be created from HEX codes too: `my_color = lv_color_hex(0x288ACF)` or `my_color = lv_folro_hex3(0x28C)`.

Mixing two colors is possible with `mixed_color = lv_color_mix(color1, color2, ratio)`. Ration can be 0..255. 0 results fully color2, 255 result fully color1. 

Colors can be created with from HSV space too using `lv_color_hsv_to_rgb(hue, saturation, value)` . `hue` should be in 0..360 range, `saturation` and `value` in 0..100 range. 

### Opacity
To describe opacity the `lv_opa_t` type is created as a wrapper to `uint8_t`. Some defines are also introduced: 

- **LV_OPA_TRANSP** Value: 0, means the opacity makes the color fully transparent
- **LV_OPA_10** Value: 25, means the color covers only a little
- **LV_OPA_20 ... OPA_80** come logically
- **LV_OPA_90** Value: 229, means the color near fully covers 
- **LV_OPA_COVER** Value: 255, means the color fully covers 

You can also use the `LV_OPA_*` defines in `lv_color_mix()` as *ratio*.

### Built-in colors

The color module defines the most basic colors:

- ![#000000](https://placehold.it/15/000000/000000?text=+) `LV_COLOR_BLACK`
- ![#808080](https://placehold.it/15/808080/000000?text=+) `LV_COLOR_GRAY`
- ![#c0c0c0](https://placehold.it/15/c0c0c0/000000?text=+) `LV_COLOR_SILVER`
- ![#ff0000](https://placehold.it/15/ff0000/000000?text=+) `LV_COLOR_RED`
- ![#800000](https://placehold.it/15/800000/000000?text=+) `LV_COLOR_MARRON`
- ![#00ff00](https://placehold.it/15/00ff00/000000?text=+) `LV_COLOR_LIME`
- ![#008000](https://placehold.it/15/008000/000000?text=+) `LV_COLOR_GREEN`
- ![#808000](https://placehold.it/15/808000/000000?text=+) `LV_COLOR_OLIVE`
- ![#0000ff](https://placehold.it/15/0000ff/000000?text=+) `LV_COLOR_BLUE`
- ![#000080](https://placehold.it/15/000080/000000?text=+) `LV_COLOR_NAVY`
- ![#008080](https://placehold.it/15/008080/000000?text=+) `LV_COLOR_TAIL`
- ![#00ffff](https://placehold.it/15/00ffff/000000?text=+) `LV_COLOR_CYAN`
- ![#00ffff](https://placehold.it/15/00ffff/000000?text=+) `LV_COLOR_AQUA`
- ![#800080](https://placehold.it/15/800080/000000?text=+) `LV_COLOR_PURPLE`
- ![#ff00ff](https://placehold.it/15/ff00ff/000000?text=+) `LV_COLOR_MAGENTA`
- ![#ffa500](https://placehold.it/15/ffa500/000000?text=+) `LV_COLOR_ORANGE`
- ![#ffff00](https://placehold.it/15/ffff00/000000?text=+) `LV_COLOR_YELLOW`

as well as `LV_COLOR_WHITE`.



## API


### Display

```eval_rst

.. doxygenfile:: lv_disp.h
  :project: lvgl
        
```

### Colors

```eval_rst

.. doxygenfile:: lv_color.h
  :project: lvgl
        
```

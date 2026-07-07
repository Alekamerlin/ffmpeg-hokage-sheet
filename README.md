# ffmpeg-hokage-sheet

The official FFmpeg doc is quite well written, so it's recommended to read it first:
```
man ffmpeg
```
In short, the logic of the FFmpeg commands is:
```
ffmpeg -global_option -input_file_option -i input_file -output_file_option output_file
```

### Crop a video

To crop a video, we can use the `-vf` key as the output file option:
```
ffmpeg -i input.mp4 -vf "crop=w:h:x:y" output.mp4
```
, where `w` and `h` are the width and height of the cropped area, and `x` and `y` are the coordinates of the upper left corner of the cropped area. All of these parameters are in pixels.

> Note: `-vf` is an alias for `-filter:v`, which means to apply a filter to the video stream.

The command above does its job on the CPU side. It's ok for short videos or low resolution videos, but otherwise it's better to use hardware acceleration to speed up the process. We can ask FFmpeg to use hardware acceleration, but we need to specify what device to use.

How to find out avialable devices? There's a command that lists all supported hardware device types on the machine:
```
ffmpeg -init_hw_device list
```
At the end of the output there will be a list of devices like:
```
...
Supported hardware device types:
videotoolbox
```
So now, that we know the device type, we can ask FFmpeg to use hardware acceleration using the `-hwaccel` global option:
```
ffmpeg -hwaccel hardware_type -i input.mp4 -vf "crop=w:h:x:y" output.mp4
```
, where `hardware_type` is the supported hardware device type. This can be `videotoolbox` for Apple M processors or `vaapi` for Intel, etc.

### Scale a video

To scale a video, we can use the same `-vf` key, but specifying that we are using a scaling filter instead of a cropping filter:
```
ffmpeg -i input.mp4 -vf "scale=w:h" output.mp4
```
, or:
```
ffmpeg -i input.mp4 -vf "scale=w:-1" output.mp4
```
, where `-1` tells FFmpeg to keep the aspect ratio of the video.

And with hardware acceleration:
```
ffmpeg -hwaccel hardware_type -i input.mp4 -vf "scale=w:-1" output.mp4
```

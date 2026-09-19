# ffmpeg-hokage-sheet

The official FFmpeg doc is quite well written, so it's recommended to read it first:
```
man ffmpeg
```
In short, the logic of the FFmpeg commands is:
```
ffmpeg -global_option -input_file_option -i input_file -output_file_option output_file
```

### Cut a video

To cut off everything after the timestamp, we can use the `-t` key as the input file option:
```
ffmpeg -t hh:mm:ss -i input.mp4 -c copy output.mp4
```
, where `hh` is the hours, `mm` is the minutes, and `ss` is the seconds in a format like 00:00:00.

> Note: the `-t` key is applied to the input file for performance, as it only reads part of the video before the timestamp, not the entire file.

> Note: the `-c copy` option is used for performance, as it tells FFmpeg not to re-encode the file. It only works without filters.

To cut off everything before the timestamp, we can use the `-ss` key as the input file option:
```
ffmpeg -ss hh:mm:ss -i input.mp4 -c copy output.mp4
```
To save a part of a video between two timestamps:
```
ffmpeg -ss hh:mm:ss -t hh:mm:ss -i input.mp4 -c copy output.mp4
```

### Scale a video

To scale a video, we can use the `-vf` key as the output file option:
```
ffmpeg -i input.mp4 -vf "scale=w:h" output.mp4
```
, where `w` and `h` are the width and height of the scaled output file. These parameters are in pixels.

Or:
```
ffmpeg -i input.mp4 -vf "scale=w:-1" output.mp4
```
, where `-1` tells FFmpeg to keep the aspect ratio of the video.

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
ffmpeg -hwaccel hardware_type -i input.mp4 -vf "scale=w:-1" output.mp4
```
, where `hardware_type` is the supported hardware device type. This can be `videotoolbox` for Apple M processors or `vaapi` for Intel, etc.

### Crop a video

To crop a video, we can use the same `-vf` key, but specifying that we are using a cropping filter instead of a scaling filter:
```
ffmpeg -i input.mp4 -vf "crop=w:h:x:y" output.mp4
```
, where `w` and `h` are the width and height of the cropped area, and `x` and `y` are the coordinates of the upper left corner of the cropped area. All of these parameters are in pixels.

And with hardware acceleration:
```
ffmpeg -hwaccel hardware_type -i input.mp4 -vf "crop=w:h:x:y" output.mp4
```

### Take a screenshot

To take a screenshot of a video at a timestamp, we can use the `-frames:v` option and tell FFmpeg to convert the video to an image:
```
ffmpeg -ss hh:mm:ss -i input.mp4 -frames:v 1 output.jpeg
```
To take a series of screenshots, we need to use the fps filter instead of the `-frames:v` option:
```
ffmpeg -ss hh:mm:ss -t hh:mm:ss -i input.mp4 -vf "fps=1" output%d.jpeg
```
The command will take one screenshot every second between the two timestamps.

> Note: the difference between `-frames:v 1` and `-vf "fps=1"` is that the first parameter tells FFmpeg how many frames to take in total, and the second tells how many frames to take per second.

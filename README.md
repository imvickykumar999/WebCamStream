# WebCamStream

24x7 YouTube Live Streaming this WebCam : http://185.98.0.114:8888/mjpg/video.mjpg

Background Music: `Calm River Keys.mp3`

## Stream Command

**Note**: The MJPEG stream has no audio, so we use background music (MP3 file) that loops continuously for 24/7 streaming.

### ✅ Working 24/7 Streaming Command (VERIFIED):

**Stream Properties:**
- Format: `mpjpeg` (MIME multipart JPEG)
- Resolution: 1920x1080
- Frame Rate: 25 fps
- Pixel Format: `yuvj420p` (full range) → converts to `yuv420p` (limited range)
- No timestamps: Uses `-use_wallclock_as_timestamps 1`

**Main Command (WORKING - Update stream key):**

```bash
ffmpeg -f mpjpeg -re -use_wallclock_as_timestamps 1 -fflags nobuffer -flags low_delay -thread_queue_size 2048 -i "http://185.98.0.114:8888/mjpg/video.mjpg" -stream_loop -1 -thread_queue_size 1024 -i "Calm River Keys.mp3" -map 0:v:0 -map 1:a:0 -c:v libx264 -preset veryfast -tune zerolatency -b:v 5000k -maxrate 5000k -bufsize 10000k -pix_fmt yuv420p -g 50 -r 25 -fps_mode cfr -c:a aac -b:a 128k -ar 44100 -ac 2 -f flv -flvflags no_duration_filesize "rtmp://a.rtmp.youtube.com/live2/YOUR_STREAM_KEY"
```

**Replace `YOUR_STREAM_KEY` with your YouTube stream key** (get it from YouTube Studio > Go Live)

### Lower Bitrate Version (if experiencing lagging):

```bash
ffmpeg -f mpjpeg -re -use_wallclock_as_timestamps 1 -fflags nobuffer -flags low_delay -thread_queue_size 2048 -i "http://185.98.0.114:8888/mjpg/video.mjpg" -stream_loop -1 -thread_queue_size 1024 -i "Calm River Keys.mp3" -map 0:v:0 -map 1:a:0 -c:v libx264 -preset ultrafast -tune zerolatency -b:v 4000k -maxrate 4000k -bufsize 8000k -pix_fmt yuv420p -g 50 -r 25 -fps_mode cfr -c:a aac -b:a 128k -ar 44100 -ac 2 -f flv -flvflags no_duration_filesize "rtmp://a.rtmp.youtube.com/live2/YOUR_STREAM_KEY"
```

### Alternative: Silent audio (if you prefer no background music):

```bash
ffmpeg -f mpjpeg -re -use_wallclock_as_timestamps 1 -fflags nobuffer -flags low_delay -thread_queue_size 2048 -i "http://185.98.0.114:8888/mjpg/video.mjpg" -f lavfi -i anullsrc=channel_layout=stereo:sample_rate=44100 -map 0:v:0 -map 1:a:0 -c:v libx264 -preset veryfast -tune zerolatency -b:v 5000k -maxrate 5000k -bufsize 10000k -pix_fmt yuv420p -g 50 -r 25 -fps_mode cfr -c:a aac -b:a 128k -ac 2 -ar 44100 -f flv -flvflags no_duration_filesize "rtmp://a.rtmp.youtube.com/live2/YOUR_STREAM_KEY"
```

### Command Parameters Explained:

**Input Options:**
- `-f mpjpeg`: Specifies MIME multipart JPEG format (correct format for this stream)
- `-re`: Read input at native frame rate (25 fps) - important for live streaming
- `-use_wallclock_as_timestamps 1`: **Required** - Stream has no timestamps, uses system clock
- `-fflags nobuffer -flags low_delay`: Reduces buffering and latency for live streaming
- `-thread_queue_size 2048`: Large buffer for video stream (prevents blocking)
- `-stream_loop -1`: Loops audio file infinitely for 24/7 streaming
- `-thread_queue_size 1024`: Buffer for audio stream

**Video Encoding:**
- `-map 0:v:0 -map 1:a:0`: Explicit stream mapping (video from input 0, audio from input 1)
- `-c:v libx264`: H.264 video codec (YouTube requirement)
- `-preset veryfast`: Encoding speed preset (faster = less CPU, lower quality; use `ultrafast` if lagging)
- `-tune zerolatency`: Optimizes for live streaming with minimal delay
- `-b:v 5000k`: Target video bitrate (5000 kbps)
- `-maxrate 5000k -bufsize 10000k`: Maximum bitrate and buffer (bufsize = 2x maxrate)
- `-pix_fmt yuv420p`: Pixel format (converts from yuvj420p to yuv420p for YouTube)
- `-g 50`: GOP size (keyframe interval) - 50 frames = 2 seconds at 25fps
- `-r 25`: Force output frame rate to 25 fps
- `-fps_mode cfr`: Constant frame rate (replaces deprecated `-vsync cfr` in FFmpeg 7.x)

**Audio Encoding:**
- `-c:a aac`: AAC audio codec (YouTube requirement)
- `-b:a 128k`: Audio bitrate (128 kbps - YouTube minimum)
- `-ar 44100`: Audio sample rate (44.1 kHz)
- `-ac 2`: Stereo audio (2 channels)

**Output Options:**
- `-f flv`: FLV container format (required for RTMP)
- `-flvflags no_duration_filesize`: Helps with continuous/24/7 streaming

### Troubleshooting Lagging Issues:

**If the stream is lagging or buffering:**
1. **Lower the bitrate**: Change `-b:v 5000k -maxrate 5000k -bufsize 10000k` to `-b:v 4000k -maxrate 4000k -bufsize 8000k`
2. **Use faster preset**: Change `-preset veryfast` to `-preset ultrafast` (less CPU usage, lower quality)
3. **Check upload speed**: Ensure your internet upload speed is at least 6-7 Mbps for 5000k video + 128k audio
4. **Check CPU usage**: High CPU usage can cause lagging - use Task Manager to monitor
5. **Reduce resolution** (if still lagging): Add `-vf scale=1280:720` before `-pix_fmt yuv420p` to stream at 720p instead of 1080p
6. **Network issues**: Use wired Ethernet instead of Wi-Fi for more stable connection

### Troubleshooting:
- **YouTube warning: "Low bitrate" or "Not receiving enough video" (INTERMITTENT)**:
  - **CRITICAL FIXES (Based on Stream Analysis)**:
    1. Use `-f mpjpeg` (not `-f mjpeg`) - Stream is MIME multipart JPEG format
    2. Use `-fps_mode cfr` instead of deprecated `-vsync cfr` (FFmpeg 7.x)
    3. Add `-fflags nobuffer -flags low_delay` to reduce latency
    4. Use `-preset ultrafast` for maximum stability
    5. Lower bitrate to 4000k initially, increase to 5000k if stable
    6. Large buffers: `thread_queue_size 2048` for video, `1024` for audio
  - **Stream Properties**: 1920x1080 @ 25fps, yuvj420p (full range) → yuv420p conversion needed
  - Ensure your upload speed is at least 5-6 Mbps for 4000k video + 128k audio
  - **Root cause**: Audio looping can block video if buffers aren't large enough
  - **Solution**: Use larger buffers, faster preset, and correct format specification
  - If still having issues, try the "Alternative: Silent audio" command to test if audio is the problem
- **YouTube warning: "Audio bitrate is 0"**:
  - **Solution**: Moved `-map` before codec settings to ensure proper stream mapping
  - Added `-r 25` to enforce constant frame rate
  - Changed `-vsync 1` to `-vsync cfr` for better compatibility
  - Ensure `-c:a aac -b:a 128k -ac 2` comes after mapping
  - **Key fix**: Map streams FIRST, then apply codec settings
- **Error -10053 (Connection aborted)**: 
  - **Most likely cause**: Invalid/expired YouTube stream key - **regenerate it in YouTube Studio**
  - Make sure stream is set to "Stream now" (not scheduled) in YouTube Studio
  - Check firewall isn't blocking RTMP (port 1935)
  - Verify network stability and upload bandwidth
- **Stream not appearing on YouTube**: Wait 10-30 seconds after starting. Check YouTube Studio > Stream Health
- **Frame drops**: Increase `-thread_queue_size` or reduce `-b:v` and `-maxrate` if your upload speed is limited
- **Deprecated pixel format warning**: This is harmless - the conversion from yuvj420p to yuv420p is handled automatically

### Important Notes:

**Before Starting:**
1. **Get your YouTube stream key**: 
   - Go to YouTube Studio > Go Live > Stream settings
   - Copy your stream key
   - Replace `YOUR_STREAM_KEY` in the command
2. **Set stream to "Stream now"** (not scheduled) in YouTube Studio
3. **Ensure MP3 file exists**: Make sure `Calm River Keys.mp3` is in the same directory or use full path

**Stream Requirements:**
- YouTube RTMP **requires an audio track** - using background music from `Calm River Keys.mp3`
- Minimum upload speed: 6-7 Mbps for stable streaming at 5000k bitrate
- The stream may take 10-30 seconds to appear on YouTube after starting
- Monitor stream health in YouTube Studio > Stream Health

**For 24/7 Streaming:**
- The audio file loops automatically with `-stream_loop -1`
- Consider using a script with auto-restart on errors for 24/7 operation
- Monitor CPU and network usage to ensure stable streaming
- Check YouTube Studio periodically for stream health warnings

**Common Issues:**
- **"I/O error"**: Invalid stream key - regenerate in YouTube Studio
- **"Not receiving video"**: Lower bitrate or check upload speed
- **Lagging**: Use `-preset ultrafast` or lower bitrate to 4000k
- **Connection aborted (-10053)**: Check stream key and ensure stream is set to "Stream now"

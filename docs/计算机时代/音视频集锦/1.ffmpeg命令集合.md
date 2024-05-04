# # ffmpeg命令集合

# 命令尝试 - (部分命令存在错误)

```bash
ffmpeg -hwaccel cuvid -c:v h264_cuvid -i input1.mp4 -hwaccel cuvid -c:v h264_cuvid -i input2.mp4 -hwaccel cuvid -c:v h264_cuvid -i input3.mp4 -filter_complex "[0:v]scale=1280:720[v0];[1:v]scale=1280:720[v1];[2:v]scale=1280:720[v2];[v0][0:a][v1][1:a][v2][2:a]concat=n=3:v=1:a=1[v][a]" -map "[v]" -map "[a]" -c:v h264_nvenc -b:v 3000k -c:a aac -b:a 192k output.mp4



ffmpeg -y -xerror -f concat -safe 0 -i filelist.txt -b:v 3000k -b:a 196k -acodec aac -y output.mp4

ffmpeg -y -f concat -safe 0 -i filelist.txt -c copy output.mp4

ffmpeg -i 50120231128165912002.mp4 -c:v libx264 -s 720x1280 -c:a aac -f mpegts 1.ts
ffmpeg -i 50120231128165912003.mp4 -c:v libx264 -s 720x1280 -c:a aac -f mpegts 2.ts
ffmpeg -i 50120231125115816001.mp4 -c:v libx264 -s 720x1280 -c:a aac -f mpegts 3.ts


ffmpeg -y -i 1.mp4 -acodec copy -vcodec copy -f mpegts 1.ts
ffmpeg -y -i 2.mp4 -acodec copy -vcodec copy -f mpegts 2.ts
ffmpeg -y -i 3.mp4 -acodec copy -vcodec copy -f mpegts 3.ts

# 不同分辨率快速合成，播放音频，画面不动

ffmpeg -y -f concat -safe 0 -i filelist.txt -b:v 3000k -b:a 196k -c:v libx264 -c:a aac output.mp4

ffmpeg -y -f concat -safe 0 -i filelist.txt -vf "scale=720:1280" -b:v 3000k -b:a 196k -c:a aac output.mp4


ffmpeg -hwaccel cuvid -c:v h264_cuvid -i input1.mp4 -hwaccel cuvid -c:v h264_cuvid -i input2.mp4 -filter_complex "[0:v] [0:a] [1:v] [1:a] concat=n=2:v=1:a=1 [v] [a]" -map "[v]" -map "[a]" -c:v h264_nvenc -b:v 3000k -c:a aac -b:a 192k output.mp4

ffmpeg -y -xerror -hwaccel cuvid -hwaccel_output_format cuda -c:v h264_cuvid -f concat -safe 0 -i 1.txt -c:v h264_nvenc -b:v 3000k -b:a 196k -c:a aac -vf "hwupload_cuda,scale_npp=720:1280" output.mp4

ffmpeg -y -xerror -hwaccel cuvid -c:v h264_cuvid -hwaccel_output_format cuda -f concat -safe 0 -i 1.txt -vf scale=720:1280 -y output.mp4


ffmpeg -y -xerror -hwaccel cuvid -f concat -safe 0 -i 1.txt -c:v h264_nvenc -b:v 3000k -b:a 196k -c:a aac -y output.mp4


ffmpeg -y -i 1.ts -i 2.ts -i 3.ts -c:v h264_nvenc -b:v 3000k -c:a aac -absf aac_adtstoasc -b:a 196k -vf "scale=720:1280" -y output.mp4


ffmpeg -c:v h264_cuvid -f concat -safe 0 -i 1.txt -c:v h264_nvenc -b:v 12000k -c:a aac -b:a 196k -vf "scale=720:1280" -y output.mp4



ffmpeg -xerror -loglevel error -f concat -safe 0 -i %s -c:v h264_nvenc -b:v 3000k -b:a 196k -c:a aac -vf scale=720:1280,format=pix_fmts=yuv420p -y %s
```

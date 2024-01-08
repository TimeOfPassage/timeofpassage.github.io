# #利用ffmpeg转换flac

# FFmpeg

官网: /[https://www.ffmpeg.org/](https://www.ffmpeg.org/ "https://www.ffmpeg.org/")

# Demo

通过python将目录下存在的flac音乐转为mp3格式

```py
import os
import shutil

ffmpeg = "D:/ffmpeg-master-latest-win64-gpl/bin/ffmpeg.exe"


# input_flac = "C:/Users/Charles.h/Downloads/music/画离弦-海伦.flac"
#
# output_mp3 = "C:/Users/Charles.h/Desktop/画离弦-海伦.mp3"

# 利用ffmpeg将flac转为mp3
# ffmpeg -i input.flac -ab 320k -map_metadata 0 -id3v2_version 3 output.mp3

# cmd = "{} -i {} -ab 320k -map_metadata 0 -id3v2_version 3 {}".format(ffmpeg, input_flac, output_mp3)
# print(cmd)
# res = os.popen(cmd)
# detail = res.readlines()
# print(detail)

def convert_flac_to_mp3(source: str, destination: str):
    cmd = "{} -i {} -ab 320k -map_metadata 0 -id3v2_version 3 {}".format(ffmpeg, source, destination)
    res = os.popen(cmd)
    res.readlines()
    return destination


def copy(source: str, destination: str):
    shutil.copy(source, destination)


def get_all_filenames(path: str) -> list[str]:
    results = os.listdir(path)
    return results


def check_dir(path: str):
    if os.path.exists(path):
        return
    os.makedirs(path)


if __name__ == "__main__":
    """
        读取所有文件，获取文件名
        如果是mp3直接复制；如果是flac，则获取文件名
        定义随机文件名通过ffmpeg转换，转换后换回原文件名
    """
    input_dir = "C:/Users/Charles.h/Downloads/music/"
    output_dir = "C:/Users/Charles.h/Desktop/music/"
    check_dir(input_dir)
    check_dir(output_dir)
    files = get_all_filenames(input_dir)
    for idx, item in enumerate(files):
        source = "{}/{}".format(input_dir, item)
        # print(idx, item)
        if item.endswith(".flac"):
            tmp_input_name = "{}/{}.flac".format(input_dir, idx)
            # copy tmp and rename file
            copy(source, tmp_input_name)
            # convert
            tmp_out_name = "{}/{}.mp3".format(input_dir, idx)
            convert_flac_to_mp3(tmp_input_name, tmp_out_name)
            # copy destination and rename
            copy(tmp_out_name, "{}/{}".format(output_dir, item.replace(".flac", ".mp3")))
            # delete tmp file
            os.remove(tmp_input_name)
            os.remove(tmp_out_name)
        else:
            copy(source, "{}/{}".format(output_dir, item))
```

‍

# #子进程执行shell命令

```py
import subprocess
import sys


def run(cmd, shell=False) -> (int, str):
    p = subprocess.Popen(cmd, shell=shell, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
    while p.poll() is None:
        line = p.stdout.readline().strip()
        if line:
            line = _decode_data(line)
            print('\033[1;35m{0}\033[0m'.format(line))
        sys.stdout.flush()
        sys.stderr.flush()
    return p.returncode


def _decode_data(byte_data: bytes):
    try:
        return byte_data.decode('UTF-8')
    except UnicodeDecodeError:
        return byte_data.decode('GB18030')


if __name__ == '__main__':
    run('docker logs -f pic-search-webclient', shell=True)
```

‍

# #SSH自动化

# ​`node.py`​​

```py
from utils.md5_utils import MD5Utils


class Node:
    def __init__(self, host: str, port: int = 22, username: str = None, password: str = None, private_key_path: str = None, root_password: str = None, desc: str = None, **kwargs):
        self.host = host
        self.port = port
        self.id = MD5Utils.create_md5(f"{host}:{port}")
        self.username = username
        self.password = password
        self.root_password = root_password
        self.private_key_path = private_key_path
        self.desc = desc

    def toMap(self):
        return {
            "id": self.id,
            "host": self.host,
            "port": self.port,
            "username": self.username,
            "password": self.password,
            "root_password": self.root_password,
            "private_key_path": self.private_key_path,
            "desc": self.desc,
        }

    @staticmethod
    def fromJson(dict_obj):
        port = dict_obj['port'] if 'port' in dict_obj else 22
        node = Node(host=dict_obj['host'], port=port, username=dict_obj['username'], password=dict_obj['password'],
                        root_password=dict_obj['root_password'], private_key_path=dict_obj['private_key_path'], desc=dict_obj['desc'])
        node.id = dict_obj['id']
        return node
```

# ​`proxy_node.py`​​

```py
import os
import sys
import time

import paramiko

from entity.node import Node


class ProxyNode:
    def __init__(self, host: str, port: int = 22, username: str = None, password: str = None, private_key_path: str = None, root_password: str = None, desc: str = None, **kwargs):
        self.host = host
        self.port = port
        self.data_id = f"{host}:{port}"
        if len(kwargs) > 0:
            data = kwargs['data'] if 'data' in kwargs else {}
            current = None
            for c in data:
                node = Node.fromJson(c)
                if node.host == host and int(node.port) == port:
                    current = node.toMap()
                    break
            self.username = current['username']
            self.password = current['password']
            if 'root_password' not in current or current['root_password'] is not None:
                self.root_password = current['password']
            else:
                self.root_password = current['root_password']
            self.private_key_path = os.path.abspath(current['private_key_path']) if len(current['private_key_path']) > 0 else None
        else:
            self.username = username
            self.password = password
            if root_password is None:
                self.root_password = self.password
            else:
                self.root_password = root_password
            self.private_key_path = private_key_path
        self.desc = desc if desc is not None else host + ":" + str(port)


def print_progress(current_progress, total_progress, prefix: str = "Progress:"):
    bar_length = 100
    format_str = "{0:." + str(1) + "f}"
    percent = format_str.format(100 * (current_progress / float(total_progress)))
    filled_length = int(round(bar_length * current_progress / float(total_progress)))
    bar = '#' * filled_length + '-' * (bar_length - filled_length)
    sys.stdout.write('\r%s |%s| %s%s %s' % (prefix, bar, percent, '%', "Complete")),
    if current_progress == total_progress:
        sys.stdout.write('\n')
    sys.stdout.flush()


class SSHProxy:

    def __init__(self, servers: list, not_once_output: bool = False):
        self.not_once_output = not_once_output
        self.servers = servers
        self.ssh_client = None
        self.business_node: ProxyNode = servers[len(servers) - 1]
        self.current_host = self.business_node.host
        self.username = self.business_node.username
        self.ssh_info = {
            # "data_id": {
            #     "client": None,
            #     "channel": None,
            #     'idx': 0
            # }
        }

    def __enter__(self):
        self.ssh_client = self.create_client()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()

    @staticmethod
    def create_ssh_client(node: ProxyNode, sock=None, timeout: int = 10):
        s = paramiko.SSHClient()
        s.load_system_host_keys()
        s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        if node.private_key_path is not None:
            private_key = paramiko.RSAKey.from_private_key_file(node.private_key_path, password=node.root_password)
            s.connect(hostname=node.host, port=node.port, username=node.username, pkey=private_key, timeout=timeout, sock=sock)
        else:
            s.connect(hostname=node.host, port=node.port, username=node.username, password=node.password, timeout=timeout, sock=sock)
        return s

    def open_sftp(self) -> paramiko.SFTPClient:
        return self.ssh_client.open_sftp()

    def open_shell(self) -> paramiko.Channel:
        return self.ssh_client.invoke_shell()

    @staticmethod
    def start_new_channel(client: paramiko.SSHClient, target_host: str, target_port: int):
        return client.get_transport().open_channel("direct-tcpip", (target_host, target_port), ('localhost', 0))

    def put_file(self, client: paramiko.SFTPClient, local_path: str, remote_path: str):
        print(f"开始上传文件[{local_path}]至[{self.current_host}:{remote_path}]")
        client.put(localpath=local_path, remotepath=remote_path, callback=print_progress)

    def get_file(self, client: paramiko.SFTPClient, local_path: str, remote_path: str):
        print(f"开始下载文件[{self.current_host}:{remote_path}]至[{local_path}]")
        client.get(localpath=local_path, remotepath=remote_path, callback=print_progress)

    @staticmethod
    def exec_command(client: paramiko.SSHClient, command: str, password: str = None):
        print(f"【执行命令】:{command}")
        stdin, stdout, stderr = client.exec_command(command=command, timeout=10, get_pty=True)
        if password is not None:
            stdin.write(f"{password}\n")
            stdin.flush()
        output = ""
        for x in stdout.readlines():
            output += x
        print(output)
        return output

    def invoke_command(self, run_channel: paramiko.Channel, command: str, need_output: bool = True):
        print(f"【{self.current_host}】【执行命令】:{command}")
        run_channel.send(f"{command}\n".encode('utf-8'))
        while not run_channel.send_ready():
            time.sleep(1)
        return self.wait_output(run_channel, command=command, callback=lambda: print(), is_print=need_output)

    def switch_root(self, run_channel: paramiko.Channel, password: str = None):
        run_channel.send(f"sudo -i\n".encode('utf-8'))
        self.wait_output(run_channel, callback=lambda: print())
        if password is not None:
            run_channel.send(f"{password}\n".encode('utf-8'))
            self.wait_output(run_channel, callback=lambda: print(), is_print=False)

    def exit_root(self, run_channel: paramiko.Channel):
        run_channel.send(f"exit\n".encode('utf-8'))
        self.wait_output(run_channel)

    def wait_output(self, run_channel, command: str = None, verbose: bool = False, is_print: bool = True, callback=None):
        while not run_channel.recv_ready():
            time.sleep(1)
        result = ''
        while True:
            res = run_channel.recv(1024).decode('utf8')
            if is_print and verbose:
                sys.stdout.write(res)
                sys.stdout.flush()
            result += res
            if res.endswith('# ') or res.endswith('$ ') or res.endswith(f"[sudo] password for {self.username}: "):
                break
        if command is not None:
            if result.startswith(command + "\r\n"):
                result = result[len(command + "\r\n"):]
        results = result.split("\r\n")[: -1]
        if is_print and not verbose:
            print("\r\n".join(results), end="")
        if callback is not None:
            callback()
        return "\r\n".join(results)

    def create_client(self) -> paramiko.SSHClient:
        nodes = self.servers
        if nodes is None or len(nodes) == 0:
            raise Exception("list must not be null")
        size = len(nodes)
        for idx, node in enumerate(nodes):
            curr_node_data_id = node.data_id
            if curr_node_data_id in self.ssh_info:
                continue
            # 获取下一个节点
            if idx + 1 == size:
                next_node = None
            else:
                next_node = nodes[idx + 1]
            # 获取前一个节点channel
            prev_data_id = self.match(idx - 1)
            if prev_data_id is None:
                current_client = self.create_ssh_client(node=node)
            else:
                current_client = self.create_ssh_client(node=node, sock=self.ssh_info[prev_data_id]['channel'])
            self.ssh_info.update(**{curr_node_data_id: {'client': current_client}})
            if next_node is not None:
                self.ssh_info[curr_node_data_id].update(**{
                    'channel': self.start_new_channel(client=current_client, target_host=next_node.host, target_port=next_node.port)
                })
            self.ssh_info[curr_node_data_id].update(**{'idx': idx})
        # 取最后一个client
        ssh_client = self.ssh_info[self.match(size - 1)]['client']
        ssh_shell = ssh_client.invoke_shell()
        self.wait_output(ssh_shell, callback=lambda: print())
        return ssh_client

    def match(self, idx):
        return_key = None
        for key in self.ssh_info.keys():
            if idx == self.ssh_info[key]['idx']:
                return_key = key
                break
        return return_key

    def close(self):
        for key in self.ssh_info.keys():
            c = self.ssh_info[key]['client']
            if c is not None:
                c.close()
        self.ssh_info.clear()


def pipeline(servers: list, callback, not_once_output: bool = False):
    if servers is None or len(servers) == 0:
        print(f"===============》 没有节点, 执行失败!!!")
        return False
    node: ProxyNode = servers[len(servers) - 1]
    print(f"===============》 {node.host}:{node.port} 开始执行...")
    # --------------------------
    # Auto Execute Area
    # --------------------------
    execute_success = False
    with SSHProxy(not_once_output=not_once_output, servers=servers) as ssh_proxy:
        # --------------------------
        # 编排部署顺序 Area, 调用方自己编排
        # --------------------------
        try:
            execute_success = callback(ssh_proxy, ssh_proxy.open_sftp(), ssh_proxy.open_shell(), ssh_proxy.business_node)
            # 没有返回值，会返回None
            if execute_success is None:
                execute_success = True
        except Exception as e:
            execute_success = False
            print(e)
        # --------------------------
        # 编排部署顺序 Area, 调用方自己编排
        # --------------------------
    if execute_success:
        print()
        print(f"===============》 {node.host}:{node.port} 执行结束...")
    else:
        print(f"===============》 {node.host}:{node.port} 执行失败!!!")
        print()
    return execute_success
```

# ​`main.py`​​

​`input`​

```py
import json

from utils.file_utils import FileUtils

# 读取文件
with open('../files/data/node', 'rb') as f:
    input_setup = json.loads(FileUtils.decrypt(f.read(), key_name="../files/encrypt/password"))
```

```py
import time

from units import input_setup
from proxy.ssh_proxy import ProxyNode, pipeline

"""
    部署服务：minio
    部署类型：单点
    部署目录：/opt/kdev
    上传目录: /home/{username}
    部署流程：
        1. 链接特定机器
        2. 上传tar包，加载tar包
        3. 清理环境，新建容器
"""
# 部署路径
deploy_path = "/opt/kdev"
# 本次升级image
image = "minio/minio:latest"
# 本次升级image构建版本号
repository = image.split(":")[0]
version = image.split(":")[1]
# 自动化运行temp目录
temp_dir = "D:/xftp/minio"
# 容器服务名
container_name = "minio"
# 运行脚本（执行脚本中的密码啥的需要自己酌情修改）
script = r"""#!/bin/bash
# read input version
version=$1
# remove container
docker rm -f {container_name}
# new container
docker run -p {host}:9000:9000 --name {container_name} \
	-e MINIO_ACCESS_KEY=minioadmin \
	-e MINIO_SECRET_KEY=minioadmin \
	-v /opt/kdev/minio/data:/data \
	-v /opt/kdev/minio/config:/root/.minio \
	-d --restart=always minio/minio:$version server /data
"""


##########
# 第一步 #
#########
def deploy(ssh_proxy, sftp_client, shell, node: ProxyNode):
    home_path = f"/home/{node.username}"
    # 切换root
    ssh_proxy.switch_root(run_channel=shell, password=node.root_password)
    # 镜像检测
    count = ssh_proxy.invoke_command(run_channel=shell, command=f"docker images | grep {container_name} | grep {version} | wc -l")
    if count is None or int(count) == 0:
        # 上传tar包
        ssh_proxy.put_file(client=sftp_client, local_path=f"{temp_dir}/{container_name}_{version}.tar", remote_path=f"{home_path}/{container_name}_{version}.tar")
        # 加载tar
        ssh_proxy.invoke_command(run_channel=shell, command=f"docker load -i {home_path}/{container_name}_{version}.tar")
    # 容器检测
    count = ssh_proxy.invoke_command(run_channel=shell, command=f"docker ps -a | grep {container_name} | wc -l")
    if count is not None and int(count) > 0:
        # 删除之前容器
        ssh_proxy.invoke_command(run_channel=shell, command=f"docker rm -f {container_name}")
    # 切换到部署目录
    ssh_proxy.invoke_command(run_channel=shell, command=f"cd {deploy_path}")
    # 脚本上传及拷贝
    with open(f"{temp_dir}/start_{container_name}.sh", "wb") as f:
        f.write(script.format(container_name=container_name, host=node.host).encode(encoding="utf8"))
    ssh_proxy.put_file(client=sftp_client, local_path=f"{temp_dir}/start_{container_name}.sh", remote_path=f"{home_path}/start_{container_name}.sh")
    ssh_proxy.invoke_command(run_channel=shell, command=f"cp {home_path}/start_{container_name}.sh {deploy_path}/start_{container_name}.sh")
    # 赋权并执行脚本
    ssh_proxy.invoke_command(run_channel=shell, command=f"/bin/bash {deploy_path}/start_{container_name}.sh {version}")
    # 检测结果
    i = 15
    is_success = False
    result = None
    while i > 0:
        i -= 1
        print(f"检测服务启动...({15 - i}/15)")
        result = ssh_proxy.invoke_command(run_channel=shell, command=rf"docker logs {container_name} | head -1000;docker ps | grep {container_name}", need_output=False)
        # 完成标记
        if "Object API (Amazon S3 compatible)" in result:
            is_success = True
            break
        time.sleep(3)
    if is_success:
        print("Ok.")
    else:
        print("服务启动失败")
        print(result)


pipeline(servers=[
    ProxyNode(host="10.1.1.10", port=23236, data=input_setup),
    ProxyNode(host="10.111.10.31", port=10022, data=input_setup)
], callback=deploy)

```

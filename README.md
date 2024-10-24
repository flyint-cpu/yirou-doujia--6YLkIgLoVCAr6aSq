
在使用 VSCode 远程 SSH 连接时，可能会遇到文件权限问题导致连接失败的情况。本文将详细记录如何为 SSH 配置文件（`config`）和私钥文件（`id_rsa`）正确设置权限，从而解决 VSCode 远程连接和 SSH 无法免密登录的问题。



> **前置背景知识：[VSCode使用Remote SSH连接远程服务器教程](https://github.com)**


### 问题背景


在 VSCode 中通过 SSH 连接远程服务器时，遇到了以下两个主要问题：


1. **SSH 配置文件（`config`）权限问题**：VSCode 提示 `Everyone` 用户组对 `config` 文件的权限过高，要求只保留读取权限。


报错信息：



```
[13:14:14.179] Log Level: 2
[13:14:14.192] Remote-SSH version: remote-ssh@0.111.2024040515
[13:14:14.193] win32 x64
[13:14:14.194] SSH Resolver called for host: guiyun
[13:14:14.194] Setting up SSH remote "guiyun"
[13:14:14.197] Using commit id "d994aede3529f4d1af9eeaeb234d32fd936243e7" and quality "insider" for server
[13:14:14.199] Install and start server if needed
[13:14:15.556] Got error from ssh: spawn C:\WINDOWS\System32\WindowsPowerShell\v1.0\ssh.exe ENOENT
[13:14:15.556] Checking ssh with "C:\WINDOWS\System32\OpenSSH\ssh.exe -V"
[13:14:15.596] > OpenSSH_for_Windows_9.5p1, LibreSSL 3.8.2
[13:14:15.599] Running script with connection command: "C:\WINDOWS\System32\OpenSSH\ssh.exe" -T -D 5902 guiyun bash
[13:14:15.601] Terminal shell path: C:\WINDOWS\System32\cmd.exe
[13:14:15.845] > Bad permissions. Try removing permissions for user: \\Everyone (S-1-1-0) on file C:/Users/Administrator/.ssh/config.
> Bad owner or permissions on C:\\Users\\Administrator/.ssh/config
> 过程试图写入的管道不存在。
> ]0;C:\WINDOWS\System32\cmd.exe
[13:14:15.845] Got some output, clearing connection timeout
[13:14:17.122] "install" terminal command done
[13:14:17.122] Install terminal quit with output: ]0;C:\WINDOWS\System32\cmd.exe
[13:14:17.122] Received install output: ]0;C:\WINDOWS\System32\cmd.exe
[13:14:17.123] Failed to parse remote port from server output
[13:14:17.124] Resolver error: Error: 
	at g.Create (c:\Users\Administrator\.vscode-insiders\extensions\ms-vscode-remote.remote-ssh-0.111.2024040515\out\extension.js:2:499181)
	at t.handleInstallOutput (c:\Users\Administrator\.vscode-insiders\extensions\ms-vscode-remote.remote-ssh-0.111.2024040515\out\extension.js:2:496503)
	at t.tryInstall (c:\Users\Administrator\.vscode-insiders\extensions\ms-vscode-remote.remote-ssh-0.111.2024040515\out\extension.js:2:620043)
	at async c:\Users\Administrator\.vscode-insiders\extensions\ms-vscode-remote.remote-ssh-0.111.2024040515\out\extension.js:2:579901
	at async t.withShowDetailsEvent (c:\Users\Administrator\.vscode-insiders\extensions\ms-vscode-remote.remote-ssh-0.111.2024040515\out\extension.js:2:583207)
	at async k (c:\Users\Administrator\.vscode-insiders\extensions\ms-vscode-remote.remote-ssh-0.111.2024040515\out\extension.js:2:576866)
	at async t.resolve (c:\Users\Administrator\.vscode-insiders\extensions\ms-vscode-remote.remote-ssh-0.111.2024040515\out\extension.js:2:580578)
	at async c:\Users\Administrator\.vscode-insiders\extensions\ms-vscode-remote.remote-ssh-0.111.2024040515\out\extension.js:2:846696
[13:14:17.126] ------

```
2. **私钥文件（`id_rsa`）无法免密码登录**：SSH 提示 `id_rsa` 私钥文件权限不安全，导致无法免密码登录。


报错信息：



```
PowerShell 7.4.5
PS C:\Users\Administrator> ssh root@103.110.228.78 -p 22
Bad permissions. Try removing permissions for user: \\Everyone (S-1-1-0) on file C:/Users/Administrator/.ssh/id_rsa.
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions for 'C:\\Users\\Administrator/.ssh/id_rsa' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "C:\\Users\\Administrator/.ssh/id_rsa": bad permissions
root@103.110.228.78's password:

```


本文将介绍如何一步步解决这两个问题，以便顺利使用 VSCode 进行远程 SSH 连接。




---


### 1\. 为 SSH 配置文件 `config` 设置权限


VSCode 要求 `config` 文件的权限必须为只读权限，并且只限于当前用户和 `Everyone` 用户组读取。如果权限设置不当，VSCode 将无法正常使用远程 SSH 功能。


#### 步骤 1：检查当前权限


首先，检查 `config` 文件的当前权限设置。打开 **PowerShell**，并运行以下命令：



```
icacls "C:\Users\Administrator\.ssh\config"

```

检查输出中是否包含 `Everyone` 用户组的权限，特别关注是否有写权限（如 `(W)`）。如果 `Everyone` 拥有写权限，这可能会导致 VSCode SSH 连接失败。


#### 步骤 2：设置 `Everyone` 只读权限


VSCode 要求 `config` 文件的 `Everyone` 用户组只能拥有读取权限。


**确保移除 `Everyone` 的写权限**：如果 `Everyone` 拥有写权限，我们需要确保完全移除它。执行以下命令来移除 `Everyone` 的写权限：



```
icacls "C:\Users\Administrator\.ssh\config" /remove "Everyone"

```

**禁用权限继承**：为了确保文件不会从父目录中继承不必要的权限，我们需要禁用权限继承：



```
icacls "C:\Users\Administrator\.ssh\config" /inheritance:r

```

这将禁用继承的权限，确保 `config` 文件只使用当前手动设置的权限。


为此，你可以通过以下命令赋予 `Everyone` 只读权限：



```
icacls "C:\Users\Administrator\.ssh\config" /grant "Everyone:R"

```

#### 步骤 3：验证权限


执行完上述操作后，验证权限设置是否正确。运行以下命令：



```
icacls "C:\Users\Administrator\.ssh\config"

```

应该看到类似 `Everyone:(R)` 的输出，表示 `Everyone` 用户组只拥有读取权限，没有写权限。这意味着文件的权限配置符合 VSCode SSH 连接的要求。通过这些步骤，你可以确保 `config` 文件的权限正确配置，从而解决 VSCode 远程 SSH 连接的权限问题。


#### 步骤 4：验证远程连接命令


在完成文件权限的调整后，你可以通过 SSH 命令验证远程连接是否正常工作。确保你已经正确设置了 `config` 文件的权限并且已完成所有权限修复步骤。


##### 1\. 在终端或 PowerShell 中运行以下命令：



```
ssh root@your.ip.exam.ple -p 22

```

* `root`：你希望使用的远程服务器的用户名。
* `your.ip.exam.ple`：你要连接的服务器的 IP 地址。
* `-p 22`：连接使用的端口号，默认是 `22`，可以根据需要调整。


##### 2\. 连接成功的情况：


如果配置正确，你将看到连接提示，可能会询问你是否接受远程服务器的指纹信息，像这样：



```
The authenticity of host 'your.ip.exam.ple (your.ip.exam.ple)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no)? yes

```

输入 `yes` 后，你将被要求输入密码或使用 SSH 密钥进行免密登录。


##### 3\. 连接失败的情况：


如果仍然无法连接，请检查：


* `config` 文件中的 IP、端口、用户名是否正确。
* 私钥文件（如 `id_rsa`）的权限是否设置正确，确保只有文件所有者可以访问。
* 远程服务器的防火墙或 SSH 服务是否配置正确。


完成后，你应该能够成功通过 SSH 命令连接到远程服务器，并验证 VSCode 的 SSH 远程连接功能是否正常。




---


### 2\. 解决 SSH 私钥 `id_rsa` 文件权限过高问题


在配置 SSH 密钥免密码登录时，SSH 需要确保私钥文件的权限非常严格，只有文件所有者能访问。如果权限过于宽松（如 `Everyone` 也有权限），SSH 将拒绝使用该私钥。


#### 步骤 1：禁用继承权限


首先，我们需要禁用私钥文件的继承权限，这样文件不会从上层目录继承不必要的权限。



```
icacls "C:\Users\Administrator\.ssh\id_rsa" /inheritance:r

```

#### 步骤 2：移除 `Everyone` 的权限


确保 `Everyone` 不再拥有对私钥文件的任何权限：



```
icacls "C:\Users\Administrator\.ssh\id_rsa" /remove "Everyone"

```

#### 步骤 3：为 `Administrator` 设置读取权限


设置私钥文件的权限，让 `Administrator` 只拥有读取权限：



```
icacls "C:\Users\Administrator\.ssh\id_rsa" //grant:r "Administrator:(R)"

```

#### 步骤 4：验证权限


再次验证权限，确保 `Everyone` 已被移除，且只有 `Administrator` 拥有读取权限：



```
icacls "C:\Users\Administrator\.ssh\id_rsa"

```

输出应显示类似于：



```
C:\Users\Administrator\.ssh\id_rsa DANCIPC\Administrator:(R)

```

#### 步骤 5：验证远程连接命令


在完成文件权限的调整后，你可以通过 SSH 命令验证远程连接是否正常工作。确保你已经正确设置了 `config` 文件的权限并且已完成所有权限修复步骤。


##### 1\. 在终端或 PowerShell 中运行以下命令：



```
ssh root@your.ip.exam.ple -p 22

```

* `root`：你希望使用的远程服务器的用户名。
* `your.ip.exam.ple`：你要连接的服务器的 IP 地址。
* `-p 22`：连接使用的端口号，默认是 `22`，可以根据需要调整。


##### 2\. 连接成功的情况：


如果配置正确，你将看到连接提示，可能会询问你是否接受远程服务器的指纹信息，像这样：



```
The authenticity of host 'your.ip.exam.ple (your.ip.exam.ple)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no)? yes

```

输入 `yes` 后，你将被要求输入密码或使用 SSH 密钥进行免密登录。


##### 3\. 连接失败的情况：


如果仍然无法连接，请检查：


* `config` 文件中的 IP、端口、用户名是否正确。
* 私钥文件（如 `id_rsa`）的权限是否设置正确，确保只有文件所有者可以访问。
* 远程服务器的防火墙或 SSH 服务是否配置正确。


完成后，你应该能够成功通过 SSH 命令连接到远程服务器，并验证 VSCode 的 SSH 远程连接功能是否正常。




---


### 3\. 解决 VSCode 远程连接问题总结


在配置 SSH 时，正确的文件权限设置至关重要。VSCode 远程连接要求 SSH 配置文件 `config` 允许 `Everyone` 用户组读取权限，而 SSH 私钥文件 `id_rsa` 必须严格禁止 `Everyone` 访问。


* **SSH 配置文件（`config`）**：保留 `Everyone` 的读取权限，移除写权限。
* **SSH 私钥文件（`id_rsa`）**：移除所有用户组的权限，只保留当前用户的读取权限。


通过调整这些权限，VSCode 将能够顺利通过 SSH 进行远程连接，同时 SSH 也能够安全地使用私钥进行免密码登录。


如果你在使用 VSCode 远程 SSH 时遇到类似问题，希望本文的解决方案能够帮助你解决权限相关的错误，提升工作效率。




---


### 参考命令列表


* 检查文件权限：



```
icacls "C:\Users\Administrator\.ssh\config"

```
* 为 `Everyone` 设置只读权限：



```
icacls "C:\Users\Administrator\.ssh\config" /grant "Everyone:(R)"

```
* 禁用继承权限：



```
icacls "C:\Users\Administrator\.ssh\id_rsa" /inheritance:r

```
* 移除 `Everyone` 权限：



```
icacls "C:\Users\Administrator\.ssh\id_rsa" /remove "Everyone"

```
* 为 `Administrator` 设置读取权限：



```
icacls "C:\Users\Administrator\.ssh\id_rsa" /grant:r "Administrator:(R)"

```
* 验证远程连接命令



```
ssh root@your.ip.exam.ple -p 22

```

希望这篇博客能够帮助你顺利解决 VSCode SSH 连接权限问题并启用 SSH 密钥免密码登录功能。


 本博客参考[西部世界官网](https://tianchuang88.com)。转载请注明出处！

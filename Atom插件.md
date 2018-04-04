# Atom插件
## Nuclide  
facebook开发的插件，提供了ide的功能。目前还没有什么研究它的功能，目前觉得Add Project Folder最有用，可以把几个开发目录放到一个视图里，这样就不需要打开多个Atom。

## remote-ftp  
sftp插件，需要在项目根目录增加.ftpconfig配置文件，配置简单，实现简单上传文件功能，无成功提示，更无远程文件比较功能。
```
{
    "protocol": "sftp",
    "host": "192.168.145.37", // string - Hostname or IP address of the server. Default: 'localhost'
    "port": 22, // integer - Port number of the server. Default: 22
    "user": "", // string - Username for authentication. Default: (none)
    "pass": "", // string - Password for password-based user authentication. Default: (none)
    "promptForPass": false, // boolean - Set to true for enable password/passphrase dialog. This will prevent from using cleartext password/passphrase in this config. Default: false
    "remote": "/export/wxsq/html", // try to use absolute paths starting with /
    "agent": "", // string - Path to ssh-agent's UNIX socket for ssh-agent-based user authentication. Linux/Mac users can set "env" as a value to use env SSH_AUTH_SOCK variable. Windows users: set to 'pageant' for authenticating with Pageant or (actual) path to a cygwin "UNIX socket." Default: (none)
    "privatekey": "", // string - Absolute path to the private key file (in OpenSSH format). Default: (none)
    "passphrase": "", // string - For an encrypted private key, this is the passphrase used to decrypt it. Default: (none)
    "hosthash": "", // string - 'md5' or 'sha1'. The host's key is hashed using this method and passed to the hostVerifier function. Default: (none)
    "ignorehost": true,
    "connTimeout": 10000, // integer - How long (in milliseconds) to wait for the SSH handshake to complete. Default: 10000
    "keepalive": 10000, // integer - How often (in milliseconds) to send SSH-level keepalive packets to the server (in a similar way as OpenSSH's ServerAliveInterval config option). Set to 0 to disable. Default: 10000
    "keyboardInteractive": false, // boolean - Set to true for enable verifyCode dialog. Keyboard interaction authentication mechanism. For example using Google Authentication (Multi factor)
    "keyboardInteractiveForPass": false, // boolean - Set to true for enable keyboard interaction and use pass options for password. No open dialog.
    "watch":[ // array - Paths to files, directories, or glob patterns that are watched and when edited outside of the atom editor are uploaded. Default : []
        "./dist/stylesheets/main.css", // reference from the root of the project.
        "./dist/stylesheets/",
        "./dist/stylesheets/*.css"
    ],
    "watchTimeout":500, // integer - The duration ( in milliseconds ) from when the file was last changed for the upload to begin.
    "filePermissions":"0644" // string - Permissions for uploaded files. WARNING: if this option is set, previously set permissions on the remote are overwritten!
}
```

## markdown
- Markdown-img-paste  
可以直接粘贴截图。图片可以选择截图保存在本地，通过相对路径引用。也可以选择截图保存到服务器。如果是上传到服务器，图片会自动保存到sm.ms服务器。保存到七牛云就不推荐，还需要填写七牛云的账号信息。注意粘贴图片的快捷键control+shift+v（mac下），并非ctrl + v 设置如下。
![](https://i.loli.net/2018/04/04/5ac4e35d0da5c.png)
https://atom.io/packages/markdown-img-paste
- markdown-pdf  
转换markdownw格式文本为pdf文件，没有快捷键，稍微麻烦一点，看说明markdown-themeable-pdf更好用一点，但是一直安装不上。

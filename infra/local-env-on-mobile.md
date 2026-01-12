# How to Open Local-env on Mobile

### 1. First you need to get the local IP of the device: 

**Windows**:  _Win+R -> cmd -> ipconfig_ -> find the adapter with IPv4 Address = 192.168.1.* (* - any value) and copy this value.

**MacOS**: _command + space -> Terminal -> ifconfig_ -> find the adapter with IPv4 Address = 192.168.1.* (* - any value) and copy this value.

Further, 192.168.1.52 will be used as an example.

### 2. Change the URLs in the local-env: </br>
In the file **deploy/environments/local/values.yaml.gotmpl** change the following lines:

_hostname: "localhost"_ --> _hostname: "192.168.1.52.nip.io"_ </br> 
_baseExternalUrl: "http://localhost:30090"_ --> _baseExternalUrl: "http://192.168.1.52.nip.io:30090"_
 
(don't forget to replace _52_ with your own value)

Then enter the following command in the terminal: </br>
```
helmfile cache cleanup && helmfile --environment local --namespace local -f deploy/helmfile.yaml apply
```

On your phone, open http://192.168.1.52.nip.io:30090.

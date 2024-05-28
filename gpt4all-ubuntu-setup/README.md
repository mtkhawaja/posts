# GPT4ALL installation on Ubuntu 24.04 LTS Noble Numbat

```shell
#!/usr/bin/env bash

wget -q "https://gpt4all.io/installers/gpt4all-installer-linux.run"
chmod +x "gpt4all-installer-linux.run"
sudo apt update && sudo apt -y install libicu-dev libxcb-cursor-dev libxcb-xinerama0
# Run GPT4All Installer
./gpt4all-installer-linux.run
```

## References

- [GPT4ALL](https://gpt4all.io)
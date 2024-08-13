# 解决linux编译一些应用时出现的头文件缺失的问题

1. sudo apt install -y apt-file
2. sudo apt-file update
3. apt-file search "X11/extensions/XInput.h"
   libxi-dev: /usr/include/X11/extensions/XInput.h
4. sudo apt install libxi-dev
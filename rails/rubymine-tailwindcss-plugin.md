# RubyMine TailwindCSS插件

当用rails创建一个使用tailwindcss的项目时 `rails new -c tailwind hello`, 
使用RubyMine打开后, TailwindCSS插件一直不好用, 经过调查，发下如下方式可以解决.

1. cd到rails项目的根目录下

2. 创建package.json文件
   `echo "{}" >> package.json`

3. 创建node_modules/tailwindcss目录
   `mkdir -p node_modules/tailwindcss`

4. 使用RubyMine重新打开rails项目，观察到RubyMine会启动
`tailwindcss-language-server`

   `ps -elf | grep node`
   
   `/usr/local/lib/nodejs/node-v16.15.0-linux-x64/bin/node /home/gyb/JetBrains/RubyMine-2023.3.2/plugins/tailwindcss/server/tailwindcss-language-server`

   这时，这个TailwindCSS插件应该好使了.

5. 如果 `tailwindcss-language-server` 没有启动, 可以拷贝`config/tailwind.config.js`到rails项目的根目录

   `cp config/tailwind.config.js .` 
   
   在用RubyMine重新打开该项目，应该就可以了，在删除掉这个根目录下的
   tailwind.config.js, 之后在打开项目，好像也可用.

6. 把 `node_modules` 和 `package.json` 添加到 `.gitignore` 中.

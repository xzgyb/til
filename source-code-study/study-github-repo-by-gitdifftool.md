# 通过git difftool去学习github上的一些开源库

- 首先执行`git log --reverse`, 获得第一条提交log, 然后`git checkout -b $first_commmit_id study`, 
  基于这个第一条提交log, 创建一个学习分支.

- 学习完这个第一条提交log分支上的代码后，然后 `git checkout main`, 切换到主分支.

- 使用下面的辅助脚本, 存为一个study.rb文件

```ruby
commit_messages = []

`git log --pretty=oneline --reverse`.each_line do |line|
  hash, message = line.split
  commit_messages << [hash, message]
end

index = ARGV[0].to_i

prev_index = index - 1
prev_index = 0 if prev_index < 0

commit1, commit2 = commit_messages[prev_index][0], commit_messages[index][0]
message = commit_messages[index][1]

command = "git difftool -t meld -y #{commit1} #{commit2}"

puts command
puts message

puts(`git ls-tree #{commit2} --name-only -r`)

`#{command}`
  
```

- 然后运行 `ruby study.rb $commit_index`, 这里$commit_index为之后的提交序号, 运行完后,
  git difftool 会调用差分工具去显示不同commit之间的代码变动, 这就可以逐步递增这个$commit_index,
  进行代码的学习.

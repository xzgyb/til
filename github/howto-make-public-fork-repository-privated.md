# 如何把github公有仓库的fork变为自己的私有仓库

1. git clone --bare https://github.com/exampleuser/public-repo.git
2. cd public-repo.git
3. git push --mirror https://github.com/yourname/private-repo.git
4. cd ..
5. rm -rf public-repo.git
6. git clone https://github.com/yourname/private-repo.git

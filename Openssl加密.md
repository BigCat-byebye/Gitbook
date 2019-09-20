# 打包压缩，并删掉源文件
tar czvf 3.tar.gz. /* --remove-files

# 加密，使用des3算法
openssl des3 -salt -k 123456 -in 3.tar.gz -out 3-jiami.tar.gz

# 加密，使用aes256算法
openssl enc -aes256 -salt -k 123456 -in xxx.tar.gz -out xxx-jiami.tar.gz

# 解密，使用aes256算法
openssl enc -aes256 -d -k 123456 in xxx-jiami.tar.gz -out xxx.tar.gz

# 解密，使用des3算法
openssl des3 -d -k 125456 -in 3-jiami.tar.gz  -out 3.tar.gz

# 解打包压缩
tar xzvf 3.tar.gz

# 解密和加密是成对的，使用某种方式加密，就必须使用某种方式解密
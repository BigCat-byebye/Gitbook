1. 在`wsl`中新建脚本

   ``` shell
   cat >> /etc/init.d/start_ssh.sh << EOF
   #!/bin/bash
   /etc/init.d/ssh restart
   EOF
   chmod +x /etc/init.d/start_ssh.sh
   ```

2. 在`windows`中设置**计划任务**

   ``` shell
   新建任务计划
   1. 设置触发器“用户登录时”
   2. 设置操作命令
   	C:\Window\System32\bash.exe -c '/etc/init.d/start_ssh.sh'
   ```

   


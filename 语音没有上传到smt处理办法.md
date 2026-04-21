# 简介

JumpServer连接

![image-20260416094120834](assets/image-20260416094120834.png)

![image-20260416094145830](assets/image-20260416094145830.png)

Real-time-notify-record.sh是负责实时监控语音并上传到smt的脚本

upload-old.sh是负责重新上传某一天语音到测试环境和正式环境的脚本

sudo vim '文件名'查看文件

## Real-time-notify-record.sh

![image-20260416094506632](assets/image-20260416094506632.png)

![image-20260416094537742](assets/image-20260416094537742.png)

## upload-old.sh

![image-20260416094622617](assets/image-20260416094622617.png)

![image-20260416094638957](assets/image-20260416094638957.png)



# 处理办法语音没有上传到smt的办法

1. 检查后台脚本是否运行：ps aux | grep real-time-notify-record.sh

![image-20260416094845364](assets/image-20260416094845364.png)

异常情况是少了一条或两条，运行：

nohup ./real-time-notify-record.sh > real-time-notify-record.log 2>&1 &

再查看一下是否添加成功



2.把upload-old.sh里的日期修改成需要重新上传录音的日期，然后运行此文件，命令 -->   ./upload-old.sh
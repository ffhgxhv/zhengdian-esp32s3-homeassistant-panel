# zhengdian-esp32s3-homeassistant-panel
使用正点原子esp32s3 box，arduino制作的，应用于home assistant的，通过mqtt更新页面按钮的中控屏项目
- 支持挥手唤醒，晃动唤醒，可以添加button，switch，可变色灯，窗帘（未验证），ha自动化（未验证）到中控屏上
- 断网时改用espnow发送指令，但是需要接收设备端自行配置执行代码，可以配合esphome设备使用。
- 主页配置了天气，时间和当前室温，可以更换三种壁纸，共有三个子界面
- 可以使用mqtt，在panel/config目录下发送json，从而实现动态更新ui，免得每次更新都要刷机
- 整体ui界面由SLS绘制，可以自行去更改相应的文件
- 通过http向ha发送命令，并通过websocket更新ui状态，所以需要在ha中申请长期令牌
- 注意！！！因为我原本是想做语音助手的，所以我初始化了es8311，但是arduino的语音唤醒太难做了，所以我放弃了，但是保留了es8311的初始化，或许有人可以用它做点什么，但我没有什么想法
- 最后，因为版权原因，我提供了三张纯色图片，分别为src/ui/images中的ui_img_miku_png.c，ui_img_nagisa_png.c，ui_img_aniime_png.c，可以自行使用sls将图片转为320*240的.c文件来替换

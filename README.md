# MemApplication
## 开发TodoList
2023.03.29

两个定时任务：
+ 定时将需要推送的记忆内容以userId:memContent的形式存到redis预热
+ 定时将redis中的内容在每天22:00推送给用户

算法实现：
+ 针对短期记忆，直接在前端设置定时器完成提醒
+ 针对长期记忆，在数据库中存储复习时间列表(next_review_time_list)和下次复习时间，在定时任务一中查询与当前日期相等的记忆内容(下次复习时间==当前日期)，将其存入Redis
+ 用户上传记忆内容文件后，根据算法生成下次复习时间写入数据库
+ 用户在系统推送提醒之前打开APP进行复习，则直接从Redis中获取到相应的复习内容即可，当用户完成复习后，将Redis中的相关内容删除并更新下次复习时间，若下次复习时间越界，则直接在数据库中标注该内容完成OK不用再提醒复习
+ 用户未在规定复习时间完成，就算系统提醒了，用户就是不学，那么在定时任务一中就要加入判断，即当前复习时间大于数据库中的下次复习时间，此时应当以当前时间为起点重新生成复习时间列表并重置下次复习时间

数据库变更：
+ 记忆内容表添加 是否完成 字段，指示是否需要进行提醒复习
+ 记忆内容表添加 复习时间表 字段，指示复习时间范围
+ 记忆内容表添加 下次复习时间 字段，指示下次复习的时间点
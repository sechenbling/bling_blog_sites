---
title: Update logs
author: Lynn
date: 2022-11-20 20:00:00 +0800
last_modified_at: 2023-01-29 18:30:00 +0800
categories: [Logs, Update]
tags: [update,logs]
syntax: colorful 
pin: true
---

## Update date: 2023-01-29

### 界面展示

- home:
  ![img.png](/blog_imgs/update_logs/windows-2023-01-29-home.png)
### Update logs

>
1️⃣ 启用根据post类别和标签进行跳转和分类查询的功能, 包括根据归档和标签进行检索的查询方式.  
2️⃣ 区分post time和modified time; 增加了timeline的tabs, 可以使用timeline以post time作为顺序查看, 也可根据主页以modified time作为顺序查看.  
3️⃣ 增加Plans tabs, 用于归纳未来计划.   
4️⃣ 每一页的分页数量从5增加到8, 更好的支持竖屏模式.
{: .prompt-tip }

## Update date: 2022-11-27

### 界面展示

- home:
  ![img.png](/blog_imgs/update_logs/windows-2022-11-27-home.png)
- post:
  ![img.png](/blog_imgs/update_logs/windows-2022-11-27-post.png)

### Update logs

>
1️⃣ contact info增加了b站的联系方式, 点击右下角即可跳转到b站个人主页.修改了contact info中个人邮箱的获取逻辑, 如果电脑中有outlook会直接打开outlook并获取联系邮箱.修改了contact
info中个人邮箱的获取逻辑, 如果电脑中有outlook会直接打开outlook并获取联系邮箱.  
2️⃣ posts sharing增加了QQ,微信,b站,微博,linkedin的sharing方式, 保留twitter; 其中qq和微信不会直接跳转到分享页面, 而是通过生成分享二维码的方式进行取代.  
3️⃣ 调整了两种颜色主题的全局表现, 现在默认界面会根据打开平台进行默认设置(比如移动端使用了暗色主题,则会自动打开黑暗模式), pc端目前是默认亮色模式  
4️⃣ 增加了top bar中searching的功能, 现在可以通过检索关键字来筛选对应的文章了.  
5️⃣ 增加了右侧update list和tags trending的显示界面, 可以通过update list来获取最新发布的博客了.  
6️⃣ 增加了打开博客后右侧显示当前文章内标题的功能,可以点击右侧content中的文章标题进行定位跳转.(此外,标题也与文章内的标题进行了连接,可以直接点击文章内标题进行界面矫正)  
{: .prompt-tip }

### Known issues

>
🔍 目前移动端可能会因为横竖屏的原因出现页面展示的问题, 不过并不影响使用,甚至还挺好的...  
🐛 暗色主题表现在不同平台可能有会bug,比如帖子文本内容也被渲染成了背景色,但是文本确实展示了,只不过和背景融为一体了...  
🐛 search中右侧清空按钮存在不生效的问题,目前手动按delete而不能点击按钮一键清空  
{: .prompt-warning }

### Others

>
🔒 暂时关闭根据post类别和标签进行跳转和分类查询的功能,目前使用相关会强制转到404页面.但不影响top bar search中根据文章内容检索的功能~  
❗ 关闭根据post发布日期排序的功能,主页展示出来的已经是排序后的结果了.  
{: .prompt-danger }

## Update date: 2022-11-20

### 界面展示

- home:
  ![img.png](/blog_imgs/update_logs/windows-2022-11-20-home.png)
- post:
  ![img.png](/blog_imgs/update_logs/windows-2022-11-20-post.png)
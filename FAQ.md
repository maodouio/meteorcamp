# 常见问题

1. 为什么 meteor 命令找不到？

   		需要在meteor项目目录内运行。

   		*注意命令的输出*

2. 如何连接 MongoDB？

   		首先要启动MongoDB，其次注意MongoDB的端口；
   		推荐办法：meteor mongo

3. 有没有什么工具能生成localmarket 一样的目录结构初始项目？

		meteor create —example localmarket
		cd localmarket
		find . -type f -exec rm -rf {} \;
		tree .
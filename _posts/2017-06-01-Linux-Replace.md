# 在linux下的VIM中替换命令的格式是;[range]s/pattern/string/[c,e,g,i]
1. range:指的是范围
2. s(search):表示搜索
3. pattern：就是要被替换的字符串
4. string：将替换pattern
5. C：每次替换前询问
6. g（globe）:不询问，将做整行替换
7. e(error):不显示error
8. i:(ignore)不分大小写

eg.

1. 询问替换文本中字符串
> %s/待替换字符串/替换的字符串/c
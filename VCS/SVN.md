linux Subversion 服务器搭建

1. 安装SVN
yum install subversion

2. 查看svn版本
svnserve –version

3. 创建svn版本库(h5是版本库名称)
svnadmin create /var/www/svn/h5

4. 配置svn信息(/svn/h5/conf)
4.1 svn服务综合配置文件（/conf/authz）
— 定义组和成员
[groups]
# harry_and_sally = harry,sally
# harry_sally_and_joe = harry,sally,&joe
h5admin = h5admin,ganxz
h5web = zhaojian,luyj,yangzp
h5server = zhanglei,wangcy

— 设置组在对应目录的访问权限
[h5:/]
@h5admin=rw
[h5:/servers]
@h5server = rw
@h5web = r
[h5:/webs]
@h5web = rw
@h5server = r
4.2 用户名口令文件（/conf/passwd）
ganxz=ganxz
zhaojian=zhaojian

4.3 权限配置文件（/conf/svnserve.conf）
把以下内容注释去掉（注意前面不能有空格）
anon-access = read
auth-access = write
password-db = passwd
authz-db = authz
realm = h5

5. 启动SVN服务
svnserve -d -r /home/data/svn/ （ -d 表示守护进程， -r 表示在后台执行 /home/data/svn/ 为svn的安装目录 ）

6. 停止SVN服务
查找svn进程号：ps -ef|grep svnserve
删除进程：kill -9 27433

7. Linux 强制提交注释（/hooks/pre-commit）
7.1 拷贝pre-commit
cp pre-commit.tmpl pre-commit

7.2 修改pre-commit配置
— 注释以下三行（前面加 # ）
# $SVNLOOK log -t “$TXN” “$REPOS” | \
# grep “[a-zA-Z0-9]” > /dev/null || exit 1
# commit-access-control.pl “$REPOS” “$TXN” commit-access-control.cfg || exit 1

— 添加以下代码
LOGMSG=$SVNLOOK log -t "$TXN" "$REPOS" | grep "[a-zA-Z0-9]" | wc -c
if [ “$LOGMSG” -lt 5 ];#要求注释不能少于5个字符，您可自定义
then
echo -e “\nLog message cann’t be empty! you must input more than 5 chars as comment!.” 1>&2
exit 1
fi

7.3 修改pre-commit文件权限
chmod 777 pre-commit
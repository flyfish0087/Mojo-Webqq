#### 1. *打印到终端的日志乱码*

程序默认会自动检测终端的编码，如果你发现乱码，可能是自动检测失败，这种情况下你可以尝试手动设置下输出编码

    $client = Mojo::Webqq->new(log_encoding=>"utf8");
    
#### 2. *如何运行多个QQ账号*

使用[Controller-API](Controller-API.md)轻松实现多账号管理

如果你只是希望简单的跑起来一两个帐号，并不想或者不会使用API，可以参考如下方法：

多账号登录主要的问题是需要把每个账号的cookie等数据保存到单独的路径，避免互相影响

在客户端初始化时提供了一个account的参数用于为每个登陆的客户端设置单独的标识，这个参数并不是真正的QQ账号，可以自由定义

每个账号的代码保存到不同的pl文件中,并设置好account参数
    
##### abc.pl文件

    use Mojo::Webqq;
    my $client = Mojo::Webqq->new(account=>"abc"); 
    $client->load("ShowMsg");
    $client->run();
    
##### def.pl文件

    use Mojo::Webqq;
    my $client = Mojo::Webqq->new(account=>"def"); 
    $client->load("ShowMsg");
    $client->run();
    
单独运行abc.pl和def.pl即可

或者不想搞很多个pl文件，可以只使用一份代码，然后运行时通过环境变量`MOJO_WEBQQ_ACCOUNT`来传递account

    use Mojo::Webqq;
    my $client = Mojo::Webqq->new(); #这里不设置account参数，而是从环境变量获取
    $client->load("ShowMsg");
    $client->run();

#### 3. *如何使用github上最新的代码进行测试*

github上的代码迭代比较频繁，定期打包发布一个稳定版本上传到cpan(Perl官方库)

通过`cpanm Mojo::Webqq`在线下载或更新的都是来自cpan的稳定版本，如果你迫不及待的想要尝试github上的最新代码，

可以手动从github下载最新源码，然后在你的 `xxxx.pl` 文件的开头

通过 `use lib 'github源码解压路径/lib/'` 来指定你要使用该路径下的`Mojo::Webqq`模块

而不是之前通过cpanm安装到系统其他路径上的`Mojo::Webqq`模块，操作步骤演示：

a. 下载最新源码的zip文件 https://github.com/sjdy521/Mojo-Webqq/archive/master.zip

b. 解压master.zip到指定路径，比如Windows C盘根目录 c:/

c. 在你的perl程序开头加上 `use lib 'c:/Mojo-Webqq-master/lib';`

d. 正常执行你的程序即可

```
#!/usr/bin/env perl
use lib 'c:/Mojo-Webqq-master/lib'; #指定加载模块时优先加载的路径
use Mojo::Webqq;
my ($host,$port,$post_api);

$host = "0.0.0.0"; #发送消息接口监听地址，没有特殊需要请不要修改
$port = 5000;      #发送消息接口监听端口，修改为自己希望监听的端口
#$post_api = 'http://xxxx';  #接收到的消息上报接口，如果不需要接收消息上报，可以删除或注释此行

my $client = Mojo::Webqq->new(log_level=>"info",http_debug=>0);
$client->load("ShowMsg");
$client->load("Openqq",data=>{listen=>[{host=>$host,port=>$port}], post_api=>$post_api});
$client->run();
```
#### 4. 日志中为什么会打印很多 “504 Gateway Time-out”

你可能会在日志中看到很多类似如下的日志

`[17/01/09 16:55:45] [warn] http://d1.web2.qq.com/channel/poll2 请求失败: 504 Gateway Time-out`

这个主要是腾讯官方接口本身的问题，即便使用网页浏览器访问w.qq.com，也会碰到这种情况

好在目前看这种错误并不会影响到消息接收，可以无视

#### 5. 扫码后可以保持多长时间在线

受限于腾讯官方服务端的限制，目前扫码成功登陆后只能保持1~2天在线，登录状态失效后会强制重新扫码登录

`Openqq`插件会上报`input_qrcode`事件

也可以通过`PostQRcode`插件把登录二维码发送到指定邮箱实现手机随时随地扫码，除此之外，也没有更好的办法避免掉线

#### 6. PHP如何获取达到Openqq插件上报的json数据

由于上报的json数据属于 application/json类型，而非application/x-www-form-urlencoded类型

因此使用常规的`$_POST`的方式是行不通的（`$_POST` 只适合获取形式为 a=1&b=2&c=3 的数据形式）

需要使用`$GLOBALS['HTTP_RAW_POST_DATA']`来直接获取http请求body中携带的原始json数据

或者使用 `$http_request_body = file_get_contents('php://input');`

再通过 php提供的`json_decode` 函数将原始json字符串转换为php对应的数据结构

php相关文档说明：

http://us3.php.net/manual/en/function.json-decode.php

http://us3.php.net/manual/en/reserved.variables.httprawpostdata.php

#### 7. screen乱码的问题

解决办法：强制UTF-8模式开启,在其他命令前加上-U 即可,如

```
screen -U -S test
screen -U -r xxx

```

#### 8. 修改用户cookie目录和记录消息目录

`Mojo::Webqq` 在类构造方法new函数中提供相关配置项,详见(https://metacpan.org/pod/distribution/Mojo-Webqq/doc/Webqq.pod#new)
默认是用户cookie是保存在/tmp目录下，如果出现登陆问题时候需要手动删除/tmp
下的相关文件，可以在启动脚本中增加参数，例子：

现在启动脚本中创建两个个目录msg 和 cookie

修改启动脚本：

    use POSIX;
    my $date=strftime("%Y%m%d",localtime());
    
    my $client=Mojo::Webqq->new(
    ua_debug    =>  0,         #是否打印详细的debug信息
    log_level   => "info",     #日志打印级别
    is_update_group => 0,
    is_update_discuss => 0,
    log_path=>"msg/qqmsg-$date",
    tmpdir=>"cookie",
    login_type  =>  "qrlogin", #"qrlogin"表示二维码登录
    );

这样消息就会保存在目录msg,session文件，二维码文件就保存在cookie目录下，
以qqmsg-20171204等保存文件中tail -f qqmsg-20171204就可以实时查看消息了。

#### 9. 发送二维码到邮箱

首先通过PostQRcode模块，如果涉及其他模块也安装下`cpanm Mojo::SMTP::Client`

修改启动脚本，在插件部分，增加如下脚本：

    $client->load("PostQRcode",data=>{
        smtp    =>  'smtp.ijz.me', #邮箱的smtp地址
        port    =>  '25', #smtp服务器端口，默认25
        from    =>  'mojoqq@ijz.me', #发件人
        to      =>  '10010@qq.com', #收件人
        user    =>  'mojoqq@ijz.me', #smtp登录帐号
        pass    =>  'Mojo-Webqq123',
        tls     =>  0,      #可选，是否使用SMTPS协议，默认为0
    });

收件人和发件人按照你实际信息填写。

#### 10. 碰到 Can't locate Mojo/Webqq.pm in @INC

说明Mojo::Webqq模块没有安装成功，通常是在执行`cpanm Mojo::Webqq`安装的过程中，由于其他依赖模块安装失败导致最终`Mojo::Webqq`没有安装成功

需要逐个检查缺少哪些模块，Linux下你可以直接执行如下命令来检查模块的安装情况,并根据提示进行操作

`curl -ks "https://raw.githubusercontent.com/sjdy521/Mojo-Webqq/master/script/check_dependencies.pl" |perl -`

#### 11. 非root账号安装后无法使用问题

解决方法：

方法1、切换到root账号下重新安装使用

方法2、在非root账号下依次执行如下操作（**不要在任何命令前面加sudo**）

1）安装local::lib模块，执行命令如下：

         cpanm --local-lib=~/perl5 local::lib  && eval $(perl -I ~/perl5/lib/perl5/ -Mlocal::lib)

2）把相关环境变量写入启动文件中，执行命令如下：

         echo 'eval "$(perl -I$HOME/perl5/lib/perl5 -Mlocal::lib)"' >>~/.bashrc

#### 12. 使用账号密码的方法无法成功登录

可能的原因是，基于账号密码的登录方式，一旦登录所在地发生较大变化，则腾讯服务器可能需要你输入图片验证码，这样就很难实现自动化操作

为了避免这种情况，你需要尽量在pl脚本所在的网络中用浏览器多登录一下 http://qun.qq.com 让腾讯服务器消除登录异常的判断

你可以在服务端搭建ssh隧道，socks5代理，支持SSL转发（CONNECT方法）的http代理等方式，然后浏览器通过服务端代理访问

参考issue： https://github.com/sjdy521/Mojo-Webqq/issues/183

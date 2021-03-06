---
title: "Host_root_passwd"
date: 2020-04-29T16:09:27+08:00
draft: false
toc: false
images:
tags: 
  - tech
---
# 记一次主机root密码修改
&ensp;&ensp;&ensp;&ensp;有个需要修改远程主机root密码的需求，但是通过passwd来修改的话，就需要程序和主机进行shell的交互，比较麻烦，于是找了条命令:  

    echo password | passwd --stdin root
&ensp;&ensp;&ensp;&ensp;可以直接在root用户下修改日root密码，开始都没啥问题，直到输入了一个带`$`符号的密码，发现修改之后的密码和实际上的对不上了，意识到echo语句里面很多特殊符号是有含义的，于是开始对各种符号进行测试总结。

&ensp;&ensp;&ensp;&ensp;通过这行命令修改主机root密码的可用特殊字符:  
`@` `%` `#` `^`  `*` `-` `=` `_` `+` `?` `/` `,` `.` `:`   
&ensp;&ensp;&ensp;&ensp;不可用的的特殊字符包括：   
`!` --结果：密码会直接修改不成功   
`$` --结果：新密码能够修改成功，但是修改后的密码完全不符合预期，如果`$`后面接字母，比如`qwer$qwer`, 那么会去掉`$`和他之后的部分，真正更新的密码会是qwer；如果`$`后面接数字，比如`qwer$1234`, 那么会依照shell的语法，取对应数字下标的命令行参数，但是单独这行命令是没有定义过命令行参数的，所以`$1`表示为空被去掉了，实际上密码变成了`qwer234`   
`&`--结果：密码修改不成功，比如密码是`qwer&1234`, &符号会作为将echo指令后台运行的命令，`qwer`会被打印，`1234`将作为新的一行命令，当然运行不了  
`(`和`)`--结果：密码修改不成功  
`\`--结果：密码修改成功但是不符合预期，`\`会作为转义符，后面接不同的字符会有不同的含义，至少和预期的密码不一样  
`;`--结果：密码修改不成功，和`&`类似，echo命令运行完成打印`qwer`,`1234`作为新命令无法运行  
`<`和`>`--结果：密码修改不成功，`<`和`>`会作为输入输出重定向符号使用，可是`1234`没有含义，内容不能输出也不能输入  
***
&ensp;&ensp;&ensp;&ensp;至此大部分能用到的特殊符号已经测试过了，没有测试过的我也不打算测了，那么解决办法有两种：  
>1. 在程序里边把捕捉密码中的不可用特殊字符，返回错误信息
>2. 使用`echo 'password' | passwd --stdin root`,将密码用''包括起来，这样即便密码中含有特殊字符也能正常使用，但是这种办法需要将含有'单引号的密码屏蔽掉，因为如果这样的`pass'word`的密码也是不成功的  





## Enigma密码机简介
Enigma密码机属于对称秘钥加密算法的轮转密码的实现。在第二次世界大战中广泛应用。
## 构造
主要有4个部件：键盘、转子、反射器、显示器；

由于反射器的存在，使得该机器的加密和解密是同一个过程。

明文（密文）通过键盘输入，经过3个转子加密，反射器反射，再次经过3个转子，最终把密文（明文）显示在显示器上。如下图：

![Enigma密码机构造图](/media/Enigma/1.jpg)

## 原理
1. 反射器reflector，这里的布局需要注意对于x，y必须有reflector[x]=reflector[y];
2. 反射机制其实是为了解密更容易的进行，其实单纯的加密只需要经过3个轮子就可以了，但是这样解密就比较困难，而引入反射器之后，加密解密就是同一个过程了。这也导致密文和明文不可能是同一个字母；除非反射器中一个字母和它本身互相反射。

## 源码
https://github.com/duanxinpeng/Enigma
## 参考
https://blog.csdn.net/kyoma/article/details/51857944

https://www.pediy.com/kssd/pediy10/90402.html
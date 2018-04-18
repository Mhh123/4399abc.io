## rainbow table

彩虹表（Rainbow Tables）就是一个庞大的、针对各种可能的字母组合预先计算好的哈希值的集合，不一定是针对MD5算法的，各种算法的都有，有了它可以快速的破解密钥

## Hash (散列函数)

Hash，一般翻译做“散列”，也有直接音译为“哈希”的，就是把任意长度的[输入](https://baike.baidu.com/item/%E8%BE%93%E5%85%A5/5481954)（又叫做预映射， pre-image），通过散列算法，变换成固定长度的[输出](https://baike.baidu.com/item/%E8%BE%93%E5%87%BA)，该输出就是散列值。这种转换是一种压缩映射，也就是，散列值的空间通常远小于输入的空间，不同的输入可能会散列成相同的输出，所以不可能从散列值来确定唯一的输入值。简单的说就是一种将任意长度的消息压缩到某一固定长度的[消息摘要](https://baike.baidu.com/item/%E6%B6%88%E6%81%AF%E6%91%98%E8%A6%81)的函数。

## PBKDF2 

(Password-Based Key Derivation Function 2)

PBKDF2应用一个伪随机函数以导出密钥。导出密钥的长度本质上是没有限制的（但是，导出密钥的最大有效搜索空间受限于基本伪随机函数的结构）。

介绍

**PBKDF2** [1][ ]() ，PBKDF2简单而言就是将salted hash进行多次重复计算，这个次数是可选择的。如果计算一次所需要的时间是1微秒，那么计算1百万次就需要1秒钟。假如攻击一个密码所需的rainbow table有1千万条，建立所对应的rainbow table所需要的时间就是115天。这个代价足以让大部分的攻击者忘而生畏。

## 相比其他算法



#### bcrypt

bcrypt是专门为密码存储而设计的算法，基于[Blowfish](https://baike.baidu.com/item/Blowfish/1677776)加密算法变形而来，由Niels Provos和David Mazières发表于1999年的USENIX。

bcrypt最大的好处是有一个参数（work factor)，可用于调整计算强度，而且work factor是包括在输出的摘要中的。随着攻击者计算能力的提高，使用者可以逐步增大work factor，而且不会影响已有用户的登陆。

bcrypt经过了很多安全专家的仔细分析，使用在以安全著称的OpenBSD中，一般认为它比PBKDF2更能承受随着计算能力加强而带来的风险。bcrypt也有广泛的函数库支持，因此我们建议使用这种方式存储密码。

#### scrypt

scrypt是由著名的[FreeBSD](https://baike.baidu.com/item/FreeBSD/413712)黑客 Colin Percival为他的备份服务 Tarsnap开发的。

和上述两种方案不同，[scrypt](https://baike.baidu.com/item/scrypt/238033)不仅计算所需时间长，而且占用的内存也多，使得[并行计算](https://baike.baidu.com/item/%E5%B9%B6%E8%A1%8C%E8%AE%A1%E7%AE%97/113443)多个摘要异常困难，因此利用rainbow table进行暴力攻击更加困难。scrypt没有在生产环境中大规模应用，并且缺乏仔细的审察和广泛的函数库支持。但是，scrypt在算法层面只要没有破绽，它的安全性应该高于PBKDF2和bcrypt。

#### PBKDF2



DK = PBKDF2(P，S，c，dkLen)

可选项： RPF 基本伪随机函数(hLen表示伪随机函数输出的字节长度)

输入：
　　P 口令，一字节串
　　S 盐值，字节串

c 迭代次数，正整数

dkLen 导出密钥的指定字节长度，正整数，最大约(2^32-1)*hLen

输出： DK 导出密钥，长度dkLen字节

步骤：

1． 如果dkLen>(2^32-1)*hLen，输出“derived key too long”并停止。
　　2． 假设l是导出密钥的hLen个字节块的个数，r表示最后一个块的字节数。

l = CEIL (dkLen / hLen) ,
　　r = dkLen - (l - 1) * hLen .

这里，CEIL(x)是“ceiling”函数，即，大于或等于x的最小整数。

\4. 对于导出密钥的每一块，运用函数F于口令P、盐S、迭代次数c和块索引以计算块：

T_1 = F (P, S, c, 1) ,
　　T_2 = F (P, S, c, 2) ,
　　...
　　T_l = F (P, S, c, l) ,

这里函数F定义为基本伪随机函数PRF应用于口令P和盐S的串联和块索引i的前c次循环的异或和。

F (P, S, c, i) = U_1 \xor U_2 \xor ... \xor U_c

其中

U_1 = PRF (P, S || INT (i)) ,
　　U_2 = PRF (P, U_1) ,
　　...
　　U_c = PRF (P, U_{c-1}) .

这里，INT(i)是整数i的四字节编码，高字节在先。

3． 串联各块，抽取前dkLen字节以产生导出密钥DK：

DK = T_1 || T_2 || ... || T_l<0..r-1>

4． 输出导出密钥DK。

注意：函数F的构造遵循“belt-and-suspenders”方法。U_i次循环被递归计算以消除敌手的并行度；它们被异或到一起以减少有关递归退化到一个小的值集的担忧。




## python hashlib模块



**Hash，一般翻译做“散列”，也有直接音译为“哈希”的，就是把任意长度的[输入](https://baike.baidu.com/item/%E8%BE%93%E5%85%A5/5481954)（又叫做预映射， pre-image），通过散列算法，变换成固定长度的[输出](https://baike.baidu.com/item/%E8%BE%93%E5%87%BA)，该输出就是散列值。**





hashlib用来替换md5和sha模块，并使他们的API一致。它由OpenSSL支持，支持如下算法：

md5,sha1, sha224, sha256, sha384, sha512



```
>>> import hashlib
>>> m = hashlib.md5()	
#创建hash对象，md5:(message-Digest Algorithm 5)消息摘要算法,得出一个128位的密文
>>> print(m)
<md5 HASH object @ 0x0000000002D49148>
>>> m.update('python')
Traceback (most recent call last):
  File "<pyshell#100>", line 1, in <module>
    m.update('python')
TypeError: Unicode-objects must be encoded before hashing
>>> m.update('python'.encode("utf-8"))
>>> print(m)
<md5 HASH object @ 0x0000000002D49148>
>>> print(m.digest())	#返回摘要，作为二进制数据字符串值
b'#\xee\xebCG\xbd\xd2k\xfck~\xe9\xa3\xb7U\xdd'
>>> print(m.hexdigest())	#返回摘要，作为十六进制数据字符串值
23eeeb4347bdd26bfc6b7ee9a3b755dd
>>> print(m.digest_size)	#16，结果hash的大小,产生的散列的字节大小
16
>>> print(m.block_size)	#64，hash内部块的大小,The internal block size of the hash algorithm in bytes
64
>>> print(hashlib.md5('python'.encode("utf-8")).hexdigest())	#简略写法
23eeeb4347bdd26bfc6b7ee9a3b755dd
>>> ss = '23eeeb4347bdd26bfc6b7ee9a3b755dd'
>>> len(ss)
32
>>> m1 = m.copy()	#复制
>>> print(m)
<md5 HASH object @ 0x0000000002D49148>
>>> print(m1)
<md5 HASH object @ 0x00000000024CE7D8>
>>> print(m.hexdigest()==m1.hexdigest())
True
```


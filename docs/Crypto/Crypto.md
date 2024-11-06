# 加密解密



## 字符串表示

python

```python
str = '123456789'

print(str[:-1])		#12345678
print(str[0])		#1
print(str[-4:])		#6789
print(str[0:4])		#1234
print(str[:4])		#1234
print(str[4:])		#56789
print(str[0:9:3])	#147
```



php—substr函数

```php
<?php
echo substr("123456789",0,10);		// 123456789
echo substr("123456789",1,8);		// 23456789
echo substr("123456789",0,5);		// 12345
echo substr("123456789",6,6);		// 789
echo substr("123456789",0,-1);		// 12345678
echo substr("123456789",-10,-2);	// 1234567
echo substr("123456789",0,-6);		// 123
echo substr("123456789",-5);		// 56789
echo substr("123456789",2);			// 3456789
echo substr("123456789",-6,6);		// 456789
?>
```





## 哈希碰撞



直接去试一个区间的值是否等于他就可以了



如题:

有一个限制条件为：

```
'8b184b' === substr(md5($b),-6,6)
```



```python
import random
import hashlib
 
value = "8b184b"
while 1:
    plainText = random.randint(10**11, 10**12 - 1)
    plainText = str(plainText)
    MD5 = hashlib.md5()
    MD5.update(plainText.encode(encoding='utf-8'))
    cipherText = MD5.hexdigest()
    if cipherText[-6:]==value :
        print("碰撞成功:")
        print("密文为:"+cipherText)
        print("明文为:"+plainText)
        break
    else:
        print("碰撞中.....")
```



例如：

硬跑：

```python
import hashlib

for num in range(10**11,10**12-1):
    res = hashlib.md5(str(num).encode()).hexdigest() #sha1改为题目需要的算法
    if res[-6:] == "8b184b":   
        print(str(num)) #等待执行结束 输出结果
        print(hashlib.md5(str(num).encode()).hexdigest())   #输出加密密文
        break
```



## RSA

RSA（Rivest–Shamir–Adleman）是一种非对称加密算法，基于大数分解的数学难题，用于数据加密和数字签名。RSA算法的原理及CTF中相关问题通常包含以下几个部分：

### RSA算法原理

1. **密钥生成**：

   - 选择两个大质数 p 和 q，计算 N=p×q。N 是 RSA 模数，密钥对的核心之一。

   - 计算欧拉函数，用于确定可以使用的指数。
     $$
     ϕ(N)=(p−1)×(q−1)
     $$
     

   - 选择一个整数 eee 满足 1<e<ϕ(N) 且 e 与 ϕ(N) 互质（通常选择 e=65537）。

   - 计算 d，满足 d×e≡1mod  ϕ(N), d 是解密密钥。

2. **加密**：给定明文 M，密文 C 的计算公式是 
   $$
   C=M^e mod  N
   $$
   

3. **解密**：收到密文 C 后，明文 M 的计算公式是
   $$
   M=C^d mod  N
   $$
   

### CTF中的RSA问题及分类

CTF中的RSA问题通常分为以下几类，基于出题角度：

#### 1. **小模数攻击**

- **场景**：如果 N很小（如只有几十位），容易通过因数分解得到 p 和 q。
- **解决方法**：直接用常见的因数分解工具（如Yafu或Msieve）分解 N，得到 p 和 q，计算私钥 d。

#### 2. **共模攻击**

- **场景**：如果多个消息使用了相同的模数 N 和不同的指数 e1,e2e_1, e_2e1,e2，可以使用数学方法（如共模攻击）破解。
- **解决方法**：应用中国剩余定理（CRT）或利用低指数攻击，还原消息。

#### 3. **低加密指数攻击**

- **场景**：如果指数 eee 很小（如 e=3e = 3e=3），而明文 M 比 N 小很多时，MeM^eMe 不需要模 N 就已经够用了，直接通过开立方计算得到明文。
- **解决方法**：通过直接开方或使用 Wiener Attack 算法获得明文。

#### 4. **小明文攻击**

- **场景**：如果明文 M 本身很小（如只有几位），可能通过尝试所有可能的明文值进行破解。
- **解决方法**：对密文 C 进行暴力枚举可能的明文值，直到找到正确的 M。

#### 5. **因数分解**

- **场景**：当 N 是两个大质数的积时，直接因数分解较难。但是如果 p 和 q 的差异较小，或者 p 和 q 是特殊数（如费马数），分解会容易得多。
- **解决方法**：使用 ECM、Pollard’s rho 或者特殊数因式分解法找到 p 和 q，再计算出 d。

#### 6. **Wiener Attack**

- **场景**：当密钥指数 d 比较小（即 d<N0.25d < N^{0.25}d<N0.25），可以使用 Wiener Attack 从公钥中推导出私钥。
- **解决方法**：使用 Wiener Attack 算法，通过连续分数逼近来获得私钥 d。

#### 7. **模不互素攻击**

- **场景**：当 N1N_1N1 和 N2N_2N2 是两个不同密钥的模数，但 N1N_1N1 和 N2N_2N2 不是互质时，可通过求出 N1N_1N1 和 N2N_2N2 的最大公因数来分解 N。
- **解决方法**：直接计算两个模数的最大公因数来分解 N。

#### 8. **Bleichenbacher Attack**

- **场景**：这是对 RSA 加密填充的攻击，利用了解密填充格式的弱点。
- **解决方法**：利用解密的 padding 逻辑来反推解密过程。

这些问题类型可以帮助选手根据题目中的不同特性判断可能的攻击方法。在CTF中，RSA题目往往提供一些解题线索，例如公开的指数 e、模数 N、某些特定的明文和密文，这些信息可以帮助分析选择合适的攻击方法。





## 椭圆曲线

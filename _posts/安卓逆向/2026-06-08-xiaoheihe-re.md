---
layout: post
title: "小黑盒 App 签名算法逆向实战复现"
subtitle: "Android API 签名机制全链路逆向：RSA加密、hkey签名、BLAKE2b哈希与 Frida Hook 完整分析"
date: 2026-06-08
author: "b0t"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - Android逆向
    - 网络安全
    - 签名算法
    - Frida
    - BLAKE2b
---

{% raw %}

## phone_num

输入手机号获取验证码：输入15555555555
抓包，看登录请求和参数:
https://api.xiaoheihe.cn/account/get_login_code/?heybox_id=-1&imei=01819c666c69367a&device_info=M2012K10C&nonce=nUNsVNs6NgNe2AyO80MzedllyN7mNAHl&hkey=56A6059B&os_type=Android&x_os_type=Android&x_client_type=mobile&os_version=13&version=1.3.368&build=1012&_time=1779157535&dw=393&channel=heybox_wandoujia&x_app=heybox&time_zone=Asia/Shanghai

接口：/account/get_login_code

| 参数                                         | 含义                                                        |
| ------------------------------------------ | --------------------------------------------------------- |
| **heybox_id=-1**                           | 用户ID，-1 表示**未登录/游客状态**                                    |
| **imei=01819c666c69367a**                  | 设备标识(IMEI)，16位，大概率是 App 内部对原始 IMEI 做了 **MD5/自定义 hash** 处理 |
| **device_info=M2012K10C**                  | 设备型号，这是**小米/Redmi 手机**的型号（如 Redmi K40 系列）                 |
| **nonce=nUNsVNs6NgNe2AyO80MzedllyN7mNAHl** | 随机数/一次性字符串，配合 `hkey` 和 `_time` 做**请求签名**，防重放攻击            |
| **hkey=56A6059B**                          | **签名校验值**，由其他参数 + 密钥/盐计算得出，服务端验证请求合法性                     |
| **os_type=Android**                        | 操作系统类型                                                    |
| **x_os_type=Android**                      | 同上，可能不同 API 各有使用                                          |
| **x_client_type=mobile**                   | 客户端类型，区别于 web/PC                                          |
| **os_version=13**                          | Android 系统版本（Android 13）                                  |
| **version=1.3.368**                        | App 版本号                                                   |
| **build=1012**                             | App 构建号                                                   |
| **_time=1779157535**                       | Unix 时间戳，防重放 + 签名的一部分                |
| **dw=393**                                 | 屏幕宽度（dp），用于适配不同设备                                         |
| **channel=heybox_wandoujia**               | **渠道标识**，表示从**豌豆荚**应用商店下载的                                |
| **x_app=heybox**                           | App 标识名                                                   |
| **time_zone=Asia/Shanghai**                | 设备时区                                                      |

请求体：phone_num:
hiBa29Ajbo8Um5jibkZMITNMrsw1VcX+ooAafjaYmGP4pbibMW5Ts7jSc0oeTJO1DJyZRlqqPh0T
S9E3Y19zWmTB4uyfMTtogxDvrLtnIwheMekY/bMxyqX2EiKQh9T0JyX94kKyWTc3iabBMRCay1XA
C4CVDyWMWooNiZMPHuU= 

明显是被加密过后的
搜一下phone_num,结果很多
![Pasted image 20260521211649.png](/img/in-post/xiaoheihe-re/Pasted%20image%2020260521211649.png)

优先挑选单参数的进行验证

这边意外发现了一个密码登陆的接口，等下试一下
```java
    @jo.e
    @o("account/login/")
    z<Result<User>> Z(@jo.c("phone_num") String str, @jo.c("pwd") String str2);
```

一看19个结果全在e类中，可以知道这是一段被混淆过的 Android Retrofit API 接口代码
![Pasted image 20260521212140.png](/img/in-post/xiaoheihe-re/Pasted%20image%2020260521212140.png)
在这里可以发现上面抓包抓到的getlogin的接口
jo.c("...") │ @Field("...") 或 @Body — POST 请求体参数   
也就是说这是发送请求的地方，i1是获取登录验证码的接口
跟进i1,交叉引用发现两条
![Pasted image 20260521213035.png](/img/in-post/xiaoheihe-re/Pasted%20image%2020260521213035.png)
跟进第一条贴出i1调用
```java
    private final void d() {
        io.reactivex.disposables.a aVar;
        if (PatchProxy.proxy(new Object[0], this, changeQuickRedirect, false, 24682, new Class[0], Void.TYPE).isSupported || (aVar = this.f76494a) == null) {
            return;
        }
        aVar.c((io.reactivex.disposables.b) com.max.xiaoheihe.network.i.a().i1(com.max.xiaoheihe.utils.z.a(this.f76496c + this.f76495b)).I5(io.reactivex.schedulers.b.d()).a4(io.reactivex.android.schedulers.a.c()).J5(new a()));
    }
```
重写后：
```java
  private final void d() {
      // this.f76494a = CompositeDisposable (RxJava 订阅管理容器)
      io.reactivex.disposables.a aVar;
      if (this.f76494a == null) return;

      this.f76494a.c(                                       // disposables.add(...)
          com.max.xiaoheihe.network.i.a()                   // Retrofit API 单例
              .i1(                                           // → POST account/get_login_code/
                  com.max.xiaoheihe.utils.z.a(               //   z.a() = 加密/编码工具方法
                      this.f76496c + this.f76495b             //   countryCode + phoneNumber (如 "+86" + "13800138000")
                  )
              )
              .I5(io.reactivex.schedulers.b.d())             // .subscribeOn(Schedulers.io())       → IO线程执行
              .a4(io.reactivex.android.schedulers.a.c())     // .observeOn(AndroidSchedulers.mainThread()) → 主线程回调
              .J5(new a())                                   // .subscribe(new Observer())           → 订阅, a是内部回调类
      );
  }
```
用户点了"获取验证码"按钮后，这个方法把 国家区号 + 手机号 拼起来，用 z.a() 编码/加密后，POST 到 account/get_login_code/，在 IO
  线程发起请求，在主线程处理回调。

  z.a() 大概率是把手机号做了一层编码（Base64/MD5/自定义算法），防止明文传输。

可以明显的看到，z.a()是加密的主要的逻辑，我们来hook一下他的返回值验证
```
[Android Emulator 5554::com.max.xiaoheihe ]-> input: <class: com.max.xiaoheihe.utils.z>
retval: U0fy/k7zXYev/QDHvuMep0j+QWxVnIy9SZ2JFAofrIm71DDFKLOWSd2sQa18BQ+Cxwyj1zG690xQ
bQqDWgun5OBoVc/j9DR7LniRA8qWD3qIvY8/1d61/OWj6F+iY/yH3c+CkvdDCoC4vatv1FzhrujH
pWv/X2X9FoIE6GA84kI=
```

hook脚本：
```js
Java.perform(function main(){
    var Z = Java.use("com.max.xiaoheihe.utils.z");
    var ret = Z.a.overload("java.lang.String").call(Z,"+8615555555555");
    console.log("input:",Z);
    console.log("retval:",ret);
});

setImmediate(main);
```

z.a跟进去就是一个RSA
```java
    public static String a(String str) {
        PatchProxyResult patchProxyResultProxy = PatchProxy.proxy(new Object[]{str}, null, changeQuickRedirect, true, 51509, new Class[]{String.class}, String.class);
        if (patchProxyResultProxy.isSupported) {
            return (String) patchProxyResultProxy.result;
        }
        try {
            byte[] bytes = str.getBytes();
            RSAPublicKey rSAPublicKeyB = b(NDKTools.getrsakey(HeyBoxApplication.M(), sc.a.f141221l, sc.a.f141227m0));
            Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
            cipher.init(1, rSAPublicKeyB);
            return Base64.encodeToString(cipher.doFinal(bytes), 0);
        } catch (Exception unused) {
            return null;
        }
    }
```

## hkey
### java层

 hkey很可能是**签名校验值**，由其他参数 + 密钥/盐计算得出，服务端验证请求合法性

找到package com.max.xiaoheihe.utils  下的p0.N()方法，hkey在这里替换生成
```java
    public static String N() {
        PatchProxyResult patchProxyResultProxy = PatchProxy.proxy(new Object[0], null, changeQuickRedirect, true, 51739, new Class[0], String.class);
        return patchProxyResultProxy.isSupported ? (String) patchProxyResultProxy.result : "hey".replaceAll("e", "ke");
    }
```
交叉引用，发现包中有两个地方调用，先看第一个在p.e的
![Pasted image 20260522141230.png](/img/in-post/xiaoheihe-re/Pasted%20image%2020260522141230.png)
```java
  map2.put(
      N(),                                                // Map 的 key
      SecurityTool.getVA(                                  // 安全工具类的方法,生成签名/校验值
          HeyBoxApplication.M(),                          //全局 Application Context
          vd2                                              // 动态参数(可能是 nonce 或时间戳)
      )
  );
```
N()生成hkey,后面那部分就是负责签名的生成
  - key = N() = "hkey"
  - value = SecurityTool.getVA(...) = 算出来的 8 位 hex 签名字符串,M() 是它的静态单例 getter
com.max.xiaoheihe.app下的HeyBoxApplication这个 App 的 Application 对象
```java
    public static HeyBoxApplication M() {
        return f75525q;
    }
```
所以HeyBoxApplication M()的作用就是就是拿到全局 Application Context
```java
String vd2 = SecurityTool.getVD(HeyBoxApplication.M(), SecurityTool.getVX(HeyBoxApplication.M(), "PAENEHAMGACOBHIEMIHIJLKJPMMHJMMQABCNGBPPENCENP"), str2, m0.j());
```
vd2像是生成的某个参数，这个放到后面分析
```java
    @m
    @k
    public static final native String getVA(@l Context context, @k String str);
```
com.max.security下的getVA方法放在了native，传入app环境和一个字符串，返回一个字符串


Context 可以拿到两个层面的信息：
  App 层面
  context.getPackageName()        → "com.max.xiaoheihe"
  context.getPackageCodePath()    → /data/app/.../base.apk 的路径
  context.getPackageManager()     → 读 APK 签名、版本号
  context.getSharedPreferences()  → 读本地存储的配置
  context.getFilesDir()           → 内部文件目录
  context.getAssets()             → assets 资源
  设备环境层面
  context.getSystemService()      → WifiManager, TelephonyManager 等
  Build.MODEL                     → "M2012K10C"（红米 K40）
  Build.VERSION.RELEASE           → "13"（Android 13）
  Build.FINGERPRINT               → 系统指纹

在com.max.security.SecurityTool中
```java
    static {
        System.loadLibrary("hbsecurity");
    }
```
可以看到加载的so文件的名称现在参数
分析完了，我们看一下map2的去向
在p.e中，将各种参数加入map2
```java
        if ("1".equals(com.max.hbcache.c.j(sc.a.N0))) {
            map2.put(I(), com.max.xiaoheihe.utils.f.X());
        }
        map2.put(J(), Build.MODEL);
        map2.put(Q(), "Android");
        map2.put(F(), Build.VERSION.RELEASE.trim());
        map2.put(W(), "Android");
        map2.put(V(), "mobile");
        map2.put(U(), g0());
        map2.put(R(), com.max.xiaoheihe.utils.f.A0());
        map2.put(G(), com.max.xiaoheihe.a.f75169g);
        map2.put(P(), com.max.hbutils.utils.z.E());
        if (strC.endsWith("/")) {
            strC = strC.substring(0, strC.length() - 1);
        }
        String str4 = strC + "/";
        SecurityTool.setKN(str2, vd2);
        SecurityTool.setKB(str4, vd2);
        SecurityTool.setKM(str2, vd2);
        map2.put(O(), str2);
        map2.put(L(), vd2);
        NDKTools.encode(HeyBoxApplication.M(), str4, str2, vd2);
        map2.put(N(), SecurityTool.getVA(HeyBoxApplication.M(), vd2));
        map2.put(H(), com.max.xiaoheihe.utils.f.w0());
        return f(str, map2);
```
最后调用f发送请求
```java
public static String f(String str, Map<String, String> map) {
        PatchProxyResult patchProxyResultProxy = PatchProxy.proxy(new Object[]{str, map}, null, changeQuickRedirect, true, 51685, new Class[]{String.class, Map.class}, String.class);
        if (patchProxyResultProxy.isSupported) {
            return (String) patchProxyResultProxy.result;
        }
        if (com.max.hbcommon.utils.c.u(str) || map == null || map.size() <= 0) {
            return str;
        }
        try {
            return q0(Uri.parse(str), map).toString();
        } catch (Exception e10) {
            e10.printStackTrace();
            return str;
        }
    }
```
那么现在我们返回来看hkey的内容：vd2的整个加密流程
```shell
● vd2 加密流程全链路

  第一步：准备四个入参
     
  vd2 = SecurityTool.getVD(context, seed, _time, userId)

  ┌────────┬───────────────────────────────────────────────┬──────────────────┐
  │  参数  │                     来源                      │      值示例      │
  ├────────┼───────────────────────────────────────────────┼──────────────────┤
  │ contex │ HeyBoxApplication.M()                         │ Application      │
  │ t      │                                               │ 上下文           │
  ├────────┼───────────────────────────────────────────────┼──────────────────┤
  │        │ SecurityTool.getVX(context, "PAENEHAMGACOBHIE │ 硬编码32字节常量 │
  │ seed   │ MIHIJLKJPMMHJMMQABCNGBPPENCENP")              │ 经 native        │
  │        │                                               │ 处理后的结果     │
  ├────────┼───────────────────────────────────────────────┼──────────────────┤
  │ _time  │ (System.currentTimeMillis() / 1000) + ""      │ "1760637928"     │
  ├────────┼───────────────────────────────────────────────┼──────────────────┤
  │ userId │ m0.j() → i().getAccount_detail().getUserid()  │ "123456" 或      │
  │        │                                               │ "-1"(未登录)     │
  └────────┴───────────────────────────────────────────────┴──────────────────┘

  注意： 第四个参数 m0.j() 不是随机数，是当前登录用户的 uid。未登录时为 "-1"。

  ---
  第二步：getVD 在 native 层计算

  getVD 进入 libhbsecurity.so，结合 context + seed + 时间戳 +
  uid，产出一个中间签名字符串 vd2。

  ---
  第三步：vd2 被五个地方消费

  vd2 (native 计算完成，回到 Java)
   │
   ├─[1] SecurityTool.setKN(_time,  vd2)     // 设置 KeyN
   ├─[2] SecurityTool.setKB(path,   vd2)     // 设置 KeyB  (path =
  "/account/data_report/")
   ├─[3] SecurityTool.setKM(_time,  vd2)     // 设置 KeyM
   │
   ├─[4] map.put("nonce", vd2)               // vd2 直接作为 URL 参数 nonce 的值
   │
   ├─[5] NDKTools.encode(context, path, _time, vd2)   // libnative-lib.so 预处理
   │
       └─[6] hkey = SecurityTool.getVA(context, vd2)      //   最终计算
          └─ map.put("hkey", hkey)                     // → URL 参数 hkey=97E8D3CB
```
从这段逻辑可以看出，vd2是用户app，seed，时间戳，userid通过SecurityTool.getVD的签名，然后vd2 直接作为 URL 参数 nonce 的值，hkey是将参数nonce拿过来进行NDKTools.encode和SecurityTool.getVA，进行预处理和最后计算


现在我们进入native层来找hkey的加密方法
### native层
我们ida打开libnative-lib.so，跟进JNI_OnLoad函数
```C
jint JNI_OnLoad(JavaVM *vm, void *reserved)
{
  jint n65540; // w19
  __int64 v3; // x20
  __int64 v4; // x0
  _QWORD v6[2]; // [xsp+0h] [xbp-10h] BYREF

  v6[1] = *(_QWORD *)(_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2)) + 40);
  v6[0] = 0LL;
  n65540 = 65540;
  if ( (*vm)->GetEnv(vm, (void **)v6, 65540LL) )
    return -1;
  v3 = v6[0];
  v4 = (*(__int64 (__fastcall **)(_QWORD, const char *))(*(_QWORD *)v6[0] + 48LL))(
         v6[0],
         "com/max/xiaoheihe/utils/NDKTools");
  if ( v4 )
    (*(void (__fastcall **)(__int64, __int64, char **, __int64))(*(_QWORD *)v3 + 1720LL))(
      v3,
      v4,
      off_64E0,                                 // "checkSignature"
      3LL);
  return n65540;
}
```
看到这里方法是动态注册的，跟进off_64E0
```C
.data:00000000000064E0 off_64E0        DCQ aChecksignature     ; DATA XREF: sub_2F5C+2C↑o
.data:00000000000064E0                                         ; sub_2F5C+34↑o ...
.data:00000000000064E0                                         ; "checkSignature"
.data:00000000000064E8                 DCQ aLjavaLangObjec     ; "(Ljava/lang/Object;)I"
.data:00000000000064F0                 DCQ _ooo_y1yl
.data:00000000000064F8                 DCQ aEncode             ; "encode"
.data:0000000000006500                 DCQ aLjavaLangObjec_1   ; "(Ljava/lang/Object;Ljava/lang/String;Lj"...
.data:0000000000006508                 DCQ _o00_y2y1
.data:0000000000006510                 DCQ aGetrsakey          ; "getrsakey"
.data:0000000000006518                 DCQ aLjavaLangObjec_0   ; "(Ljava/lang/Object;Ljava/lang/String;Lj"...
.data:0000000000006520                 DCQ _oOo_y2yl
.data:0000000000006520 ; .data         ends
```
发现是一个注册表的结构体，具体参照下面
```shell
  JNINativeMethod 结构体每条占 3 个 QWORD（24 字节）：
  struct JNINativeMethod {
      char *name;       // Java 方法名
      char *signature;  // 方法签名
      void *fnPtr;      // native 函数指针
  };
```
那么encode就对应的是_o00_y2y1，跟进
伪代码如下：
```C
__int64 __fastcall o00_y2y1(__int64 *a1, __int64 a2, __int64 context, __int64 path, __int64 time, __int64 vd2)
{
  const char *path_; // x22
  const char *time_; // x23
  const char *vd2_; // x24
  __int64 result; // x0
  int v14; // w0
  int vd2len; // w26
  __int64 v16; // x8
  _BYTE *v17; // x9
  char v18; // w10
  int v19; // t1
  unsigned int n0x1A; // w12
  char v21; // w13
  int get_timestamp__; // w0
  size_t v23; // x0
  char *s_2; // x24
  unsigned int pathlen; // w0
  size_t n0x41; // x0
  unsigned int v27; // w15
  unsigned int v28; // w13
  unsigned int v29; // w15
  unsigned __int64 v30; // x10
  unsigned int v31; // w13
  char v32; // w14
  unsigned int v33; // w9
  unsigned int v34; // w11
  unsigned int v35; // w8
  int n10; // w24
  __int64 v37; // x1
  __int64 v38; // x2
  char n48_1; // w8
  char n48_2; // w10
  __int64 v41; // x26
  unsigned int v42; // w20
  __int64 v43; // x21
  __int64 v44; // x22
  __int64 v45; // x23
  __int64 v46; // x24
  int32x4_t v47; // [xsp+0h] [xbp-180h] BYREF
  char _1234567[12]; // [xsp+10h] [xbp-170h] BYREF
  __int16 n48; // [xsp+1Ch] [xbp-164h] BYREF
  char v50; // [xsp+1Eh] [xbp-162h]
  __int128 a1a; // [xsp+20h] [xbp-160h] BYREF
  __int128 v52; // [xsp+30h] [xbp-150h]
  __int128 v53; // [xsp+40h] [xbp-140h]
  __int128 v54; // [xsp+50h] [xbp-130h]
  __int128 v55; // [xsp+60h] [xbp-120h]
  __int128 v56; // [xsp+70h] [xbp-110h]
  __int128 v57; // [xsp+80h] [xbp-100h]
  __int128 v58; // [xsp+90h] [xbp-F0h]
  __int128 v59; // [xsp+A0h] [xbp-E0h]
  __int128 v60; // [xsp+B0h] [xbp-D0h]
  __int128 v61; // [xsp+C0h] [xbp-C0h]
  __int128 v62; // [xsp+D0h] [xbp-B0h]
  __int128 v63; // [xsp+E0h] [xbp-A0h]
  __int128 v64; // [xsp+F0h] [xbp-90h]
  __int128 v65; // [xsp+100h] [xbp-80h]
  __int128 v66; // [xsp+110h] [xbp-70h]
  int src_; // [xsp+128h] [xbp-58h] BYREF
  char v68; // [xsp+12Ch] [xbp-54h]
  char v69; // [xsp+12Dh] [xbp-53h]
  char bytes_to_int32___; // [xsp+12Eh] [xbp-52h]
  char v71; // [xsp+12Fh] [xbp-51h]
  _BYTE _2345JKMNPQRT6789BCDFGHVWXY[26]; // [xsp+130h] [xbp-50h] BYREF
  _BYTE v73[46]; // [xsp+14Ah] [xbp-36h] BYREF
  __int64 v74; // [xsp+178h] [xbp-8h]

  v74 = *(_QWORD *)(_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2)) + 40);
  path_ = (const char *)(*(__int64 (__fastcall **)(__int64 *, __int64, _QWORD))(*a1 + 1352))(a1, path, 0LL);
  time_ = (const char *)(*(__int64 (__fastcall **)(__int64 *, __int64, _QWORD))(*a1 + 1352))(a1, time, 0LL);
  vd2_ = (const char *)(*(__int64 (__fastcall **)(__int64 *, __int64, _QWORD))(*a1 + 1352))(a1, vd2, 0LL);
  result = 0LL;
  if ( path_ && time_ && vd2_ )
  {
    qmemcpy(_2345JKMNPQRT6789BCDFGHVWXY, "2345JKMNPQRT6789BCDFGHVWXY", sizeof(_2345JKMNPQRT6789BCDFGHVWXY));
    v73[32] = 0;
    v14 = strlen(vd2_);
    if ( v14 < 1 )
    {
      vd2len = 0;
    }
    else
    {
      vd2len = 0;
      v16 = (unsigned int)v14;
      v17 = v73;
      do
      {
        v19 = *(unsigned __int8 *)vd2_++;
        v18 = v19;
        n0x1A = v19 - 97;
        v21 = v19 - 32;
        if ( (unsigned int)(v19 - 48) < 10 )
          ++vd2len;
        if ( n0x1A < 26 )
          v18 = v21;
        --v16;
        *v17++ = v18;
      }
      while ( v16 );
    }                                           // convert vd2's lowercase to uppercase
                                                // 
                                                // 
    get_timestamp__ = atoi(time_);              // get timestamp
                                                // 
                                                // 
    dword_6590 = (unsigned __int8)((unsigned int)(get_timestamp__ + vd2len) >> 16);
    dword_6594 = (unsigned __int8)((unsigned __int16)(get_timestamp__ + vd2len) >> 8);
    v71 = get_timestamp__ + vd2len;
    src_ = 0;
    v65 = 0u;
    v66 = 0u;
    v63 = 0u;
    v64 = 0u;
    v61 = 0u;
    v62 = 0u;
    v59 = 0u;
    v60 = 0u;
    v57 = 0u;
    v58 = 0u;
    v55 = 0u;
    v56 = 0u;
    v53 = 0u;
    v54 = 0u;
    dword_658C = (unsigned int)(get_timestamp__ + vd2len) >> 24;
    dword_6598 = (unsigned __int8)(get_timestamp__ + vd2len);
    v68 = (unsigned int)(get_timestamp__ + vd2len) >> 24;
    v69 = (unsigned int)(get_timestamp__ + vd2len) >> 16;
    bytes_to_int32___ = (unsigned __int16)(get_timestamp__ + vd2len) >> 8;// bytes to int32
                                                // 
                                                // 
                                                // 
    a1a = 0u;
    v52 = 0u;
    v23 = strlen(path_);
    s_2 = (char *)malloc((2 * (unsigned int)(((v23 + 2) * (unsigned __int128)0xAAAAAAAAAAAAAAABLL) >> 64)) & 0xFFFFFFFC | 1LL);
    pathlen = strlen(path_);
    b64((unsigned __int64)path_, pathlen, s_2); // base64 for url
                                                // 
                                                // 
                                                // 
    n0x41 = strlen(s_2);
    HMACSHA1(&a1a, (__int64)s_2, &src_, n0x41, 8uLL);// hmac  = HMAC_custom(key, path_b64) 
                                                // 
                                                // 
    v27 = *(_DWORD *)((unsigned __int64)&a1a | BYTE3(v52) & 0xF);
    dword_659C = BYTE3(v52);
    dword_65A0 = BYTE3(v52) & 0xF;
    dword_65A4 = v27;
    v28 = bswap32(v27);
    v29 = v28 & 0x7FFFFFFF;
    dword_65AC = (v28 & 0x7FFFFFFF) / 0x271F35A0;
    strcpy(_1234567, "1234567");
    dword_65A8 = v28;
    v30 = 1307386003LL * ((v28 >> 2) & 0x1FFFFFFF);
    v31 = (v28 & 0x7FFFFFFF) / 58;
    v32 = _2345JKMNPQRT6789BCDFGHVWXY[v29 - 58 * v31];
    v33 = (unsigned __int8)_2345JKMNPQRT6789BCDFGHVWXY[v31 % 58];
    LODWORD(v30) = (unsigned __int8)_2345JKMNPQRT6789BCDFGHVWXY[(v30 >> 40) % 58];
    v34 = (unsigned __int8)_2345JKMNPQRT6789BCDFGHVWXY[v29 / 0x2FA28 % 58];
    v35 = (unsigned __int8)_2345JKMNPQRT6789BCDFGHVWXY[v29 / 0xACAD10 % 58];
    v50 = 0;
    _1234567[0] = v32;
    _1234567[1] = v33;
    _1234567[2] = v30;
    _1234567[3] = v34;
    _1234567[4] = v35;
    v47.n128_u64[0] = __PAIR64__(v30, v33);
    v47.n128_u64[1] = __PAIR64__(v35, v34);
    n48 = 0;                                    // b58(hmac)
                                                // 
                                                // 
                                                // 
    sub_2CD0(&v47);                             // 用 AES 的 MixColumns 扩散逻辑搅拌前 5 个 Base58 字符的二进制表示，然后求纵向和模 100 得到校验码
                                                //   sub_2CD0(&v47);              // 对 16 字节 SIMD 寄存器做 GF(2⁸) 扩散
                                                //   n10 = vaddvq_s32(v47) % 100; // 4 个 32-bit 元素纵向求和 → 模 100
                                                //   sub_2EB8(&n48, n10);         // 转成 2 位 ASCII 数字
                                                //   _1234567[5] = n48_2;         // 十位
                                                //   _1234567[6] = n48_1;         // 个位
    n10 = vaddvq_s32(v47) % 100;
    sub_2EB8((__int64)&n48, v37, v38, (unsigned int)n10);
    n48_1 = n48;
    if ( n10 >= 10 )
      n48_2 = n48;
    else
      n48_2 = 48;
    if ( n10 >= 10 )
      n48_1 = HIBYTE(n48);
    _1234567[5] = n48_2;
    _1234567[6] = n48_1;
    (*(void (__fastcall **)(__int64 *, __int64, const char *))(*a1 + 1360))(a1, path, path_);
    (*(void (__fastcall **)(__int64 *, __int64, const char *))(*a1 + 1360))(a1, time, time_);
    v41 = *a1;
    v42 = __strlen_chk(_1234567, 8uLL);
    v43 = (*(__int64 (__fastcall **)(__int64 *, const char *))(v41 + 48))(a1, "java/lang/String");
    v44 = (*(__int64 (__fastcall **)(__int64 *, const char *))(v41 + 1336))(a1, "UTF-8");
    v45 = (*(__int64 (__fastcall **)(__int64 *, __int64, const char *, const char *))(v41 + 264))(
            a1,
            v43,
            "<init>",
            "([BLjava/lang/String;)V");
    v46 = (*(__int64 (__fastcall **)(__int64 *, _QWORD))(v41 + 1408))(a1, v42);
    (*(void (__fastcall **)(__int64 *, __int64, _QWORD, _QWORD, char *))(v41 + 1664))(a1, v46, 0LL, v42, _1234567);
    return (*(__int64 (__fastcall **)(__int64 *, __int64, __int64, __int64, __int64))(v41 + 224))(
             a1,
             v43,
             v45,
             v46,
             v44);
  }
  return result;
}
```
还原后大概是这样的
```C
/**
 * NDKTools.encode() — C 语言还原
 *
 * 基于 ARM64 汇编逆向:
 *   libnative-lib.so: _o00_y2y1 (0x3c2c)
 *   libnative-lib.so: 0x35f8  (自定义 MD5/SHA-1 混合哈希)
 *   libnative-lib.so: 0x32bc  (Base64 编码 path)
 *   libnative-lib.so: 0x34b4  (HMAC 结构)
 *
 * 算法概要:
 *   1. 预处理 vd2: 小写→大写, 统计数字数
 *   2. key32 = atoi(time) + digit_count → 分解为4字节
 *   3. path_b64 = base64_encode(path)           ← 仅对 URL path 做 Base64
 *   4. hmac  = HMAC_custom(key, path_b64)       // key = HMAC_KEY + 4字节
 *   5. rev(hmac[0]) & 0x7FFFFFFF, 连续除以58取余 → 5个Base58字符
 *      (查表 "2345JKMNPQRT6789")
 *   6. 后4个余数做 AES MixColumns (GF(2^8), 0x1B) 扩散
 *      → 纵向求和 → mod 100 → 2位校验码
 *   7. 返回 "XXXXXNN" (5个Base58字符 + 2位数字)
 *
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdint.h>
/* <ctype.h> not used directly */

/* ============================================================
 * 常量
 * ============================================================ */

/* 字符集 (libnative-lib.so .rodata 0xb30) */
static const char CHARSET[17] = "2345JKMNPQRT6789";

/* HMAC 密钥 (libnative-lib.so .rodata 0xbe5) */
static const unsigned char HMAC_KEY[] =
    "+mW6r3TSU4\n"
    "ECvNYqDMIS/bhCj2QaH5GI/KZb2TBp+CBvUj9SLFnmJQ0kzHzHoGZCQ88VevCffF\n"
    "7JePGF9cmKQqotlfTKbV4oxV5iLz7JSG6b/";

/* 模运算乘数 (从 _o00_y2y1 汇编提取) */
/* 自定义哈希 IV: 4×MD5 + 1×SHA-256 (从 0x35f8 汇编提取) */
static const uint32_t HASH_IV[5] = {
    0x67452301,  /* MD5 H0 */
    0xEFCDAB89,  /* MD5 H1 */
    0x98BADCFE,  /* MD5 H2 */
    0x10325476,  /* MD5 H3 */
    0xC3D2E1F0,  /* SHA-256 H5 (额外状态字) */
};

/* SHA-1 轮常量 (从 0x35f8 汇编提取) */
static const uint32_t SHA1_K[4] = {
    0x5A827999,  /* K0: rounds 0-19 */
    0x6ED9EBA1,  /* K1: rounds 20-39 */
    0x8F1BBCDC,  /* K2: rounds 40-59 */
    0xCA62C1D6,  /* K3: rounds 60-79 */
};

/* 全局 key 状态 (模拟 native 0x58c-0x598) */
static uint32_t g_keys[5];

/* ============================================================
 * 工具函数
 * ============================================================ */

static inline uint32_t rotl32(uint32_t n, int b)
{
    return ((n << b) | (n >> (32 - b)));
}

/* MD5 轮函数 */
static inline uint32_t F_md5(uint32_t x, uint32_t y, uint32_t z)
{
    return (x & y) | ((~x) & z);          /* (b & c) | (~b & d) */
}

static inline uint32_t G_md5(uint32_t x, uint32_t y, uint32_t z)
{
    return (x & z) | (y & (~z));          /* (d & b) | (~d & c) */
}

static inline uint32_t H_md5(uint32_t x, uint32_t y, uint32_t z)
{
    return x ^ y ^ z;                     /* b ^ c ^ d */
}

static inline uint32_t I_md5(uint32_t x, uint32_t y, uint32_t z)
{
    return y ^ (x | (~z));                /* c & (b|d) | (b&d) */
}

/* ============================================================
 * Base64 编码 (标准)
 * ============================================================ */

static int base64_encode(const unsigned char *src, int slen, char *dst)
{
    static const char *tbl =
        "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";

    int i, j = 0;
    uint32_t v;

    for (i = 0; i < slen; i += 3) {
        v  = ((uint32_t)(i < slen ? src[i]     : 0)) << 16;
        v |= ((uint32_t)(i + 1 < slen ? src[i + 1] : 0)) << 8;
        v |= ((uint32_t)(i + 2 < slen ? src[i + 2] : 0));

        dst[j++] = tbl[(v >> 18) & 0x3F];
        dst[j++] = tbl[(v >> 12) & 0x3F];
        dst[j++] = (i + 1 < slen) ? tbl[(v >> 6) & 0x3F] : '=';
        dst[j++] = (i + 2 < slen) ? tbl[v & 0x3F]        : '=';
    }
    dst[j] = '\0';
    return j;
}

/* ============================================================
 * 自定义 MD5/SHA-1 混合哈希 (函数 0x35f8)
 *
 * 结构:
 *   IV:      4×MD5 + 1×SHA-256
 *   轮函数:   MD5 F/G/H/I (各20轮, 共80轮)
 *   轮常量:   SHA-1 K0/K1/K2/K3
 *   填充:     标准 MD/SHA (0x80 + zeros + 8字节大端长度)
 *   输出:     20 字节 (5×32位, 大端)
 * ============================================================ */

static void custom_hash(const unsigned char *data, size_t len,
                        unsigned char out[20])
{
    uint32_t a, b, c, d, e;
    uint32_t aa, bb, cc, dd, ee;
    uint32_t words[16];
    size_t padded_len, total_blocks, i;
    unsigned char *padded;
    int round;
    uint32_t f_val, k_val, temp;

    /* 初始化 */
    a = HASH_IV[0];
    b = HASH_IV[1];
    c = HASH_IV[2];
    d = HASH_IV[3];
    e = HASH_IV[4];

    /* 填充: 0x80 + zeros + 8字节大端长度 (单位: bits) */
    padded_len = len + 1 + 8;
    if (padded_len % 64)
        padded_len += 64 - (padded_len % 64);

    padded = (unsigned char *)calloc(1, padded_len);
    memcpy(padded, data, len);
    padded[len] = 0x80;

    /* 8字节大端长度 */
    uint64_t bitlen = (uint64_t)len * 8;
    for (i = 0; i < 8; i++)
        padded[padded_len - 8 + i] = (bitlen >> (56 - i * 8)) & 0xFF;

    total_blocks = padded_len / 64;

    /* 逐块处理 */
    for (size_t blk = 0; blk < total_blocks; blk++) {
        const unsigned char *block = padded + blk * 64;

        /* 加载16个32位字 (小端序, 与 MD5 一致) */
        for (i = 0; i < 16; i++)
            words[i] = ((uint32_t *)block)[i];

        aa = a; bb = b; cc = c; dd = d; ee = e;

        /* 80轮 */
        for (round = 0; round < 80; round++) {
            if (round < 20) {
                f_val = F_md5(b, c, d);
                k_val = SHA1_K[0];
            } else if (round < 40) {
                f_val = H_md5(b, c, d);
                k_val = SHA1_K[1];
            } else if (round < 60) {
                f_val = I_md5(b, c, d);
                k_val = SHA1_K[2];
            } else {
                f_val = G_md5(b, c, d);
                k_val = SHA1_K[3];
            }

            temp = a + f_val + k_val + words[round % 16];
            temp = rotl32(temp, 7);

            /* SHA-1 状态轮转: a'=T, b'=a, c'=ROL(b,30), d'=c, e'=d */
            e = d;
            d = c;
            c = rotl32(b, 30);
            b = a;
            a = temp;
        }

        /* 加法更新 */
        a += aa;
        b += bb;
        c += cc;
        d += dd;
        e += ee;
    }

    free(padded);

    /* 输出20字节 (5×32位, 小端) */
    uint32_t out_words[5] = { a, b, c, d, e };
    for (i = 0; i < 5; i++) {
        out[i * 4 + 0] = out_words[i] & 0xFF;
        out[i * 4 + 1] = (out_words[i] >> 8) & 0xFF;
        out[i * 4 + 2] = (out_words[i] >> 16) & 0xFF;
        out[i * 4 + 3] = (out_words[i] >> 24) & 0xFF;
    }
}

/* ============================================================
 * HMAC (基于自定义哈希, 使用 0x36/0x5c XOR 结构)
 * 函数 0x34b4 的完整还原
 * ============================================================ */

static void hmac_custom(const unsigned char *key, int klen,
                        const unsigned char *msg, int mlen,
                        unsigned char out[20])
{
    unsigned char key_block[64];
    unsigned char ipad[64], opad[64];
    unsigned char inner_hash[20];
    unsigned char *inner_msg, *outer_msg;
    int i;

    /* 密钥处理: >64字节先哈希 */
    memset(key_block, 0, 64);
    if (klen > 64) {
        custom_hash(key, klen, key_block);
    } else {
        memcpy(key_block, key, klen);
    }

    /* ipad = key XOR 0x36, opad = key XOR 0x5c */
    for (i = 0; i < 64; i++) {
        ipad[i] = key_block[i] ^ 0x36;
        opad[i] = key_block[i] ^ 0x5c;
    }

    /* inner = hash(ipad || message) */
    inner_msg = (unsigned char *)malloc(64 + mlen);
    memcpy(inner_msg, ipad, 64);
    memcpy(inner_msg + 64, msg, mlen);
    custom_hash(inner_msg, 64 + mlen, inner_hash);
    free(inner_msg);

    /* outer = hash(opad || inner_hash) */
    outer_msg = (unsigned char *)malloc(64 + 20);
    memcpy(outer_msg, opad, 64);
    memcpy(outer_msg + 64, inner_hash, 20);
    custom_hash(outer_msg, 64 + 20, out);
    free(outer_msg);
}

/* ============================================================
 * AES MixColumns in GF(2^8)  (函数 0x2cd0)
 *
 * 对 4 个 Base58 余数做列混合扩散, 不可约多项式 x^8 + x^4 + x^3 + x + 1 (0x1B)
 * ============================================================ */

static uint8_t gf_xtime(uint8_t x)
{
    return (x & 0x80) ? (uint8_t)((x << 1) ^ 0x1B) : (uint8_t)(x << 1);
}

/* MixColumns on column [a,b,c,d], returns sum of 4 output bytes */
static uint32_t mix_checksum(uint8_t a, uint8_t b, uint8_t c, uint8_t d)
{
    /* out0 = 2*a ^ 3*b ^ 1*c ^ 1*d */
    uint8_t r0 = gf_xtime(a) ^ gf_xtime(b) ^ b ^ c ^ d;
    /* out1 = 1*a ^ 2*b ^ 3*c ^ 1*d */
    uint8_t r1 = a ^ gf_xtime(b) ^ gf_xtime(c) ^ c ^ d;
    /* out2 = 1*a ^ 1*b ^ 2*c ^ 3*d */
    uint8_t r2 = a ^ b ^ gf_xtime(c) ^ gf_xtime(d) ^ d;
    /* out3 = 3*a ^ 1*b ^ 1*c ^ 2*d */
    uint8_t r3 = gf_xtime(a) ^ a ^ b ^ c ^ gf_xtime(d);

    return (uint32_t)r0 + (uint32_t)r1 + (uint32_t)r2 + (uint32_t)r3;
}

/* ============================================================
 * NDKTools.encode() — 主函数
 * ============================================================ */

/**
 * encode - 计算签名中间值
 *
 * @param path     URL 路径, e.g. "/account/data_report/"
 * @param time_str 时间戳字符串, e.g. "1760637928"
 * @param vd2      getVD() 返回的中间签名值 (32位hex)
 * @param out       输出缓冲区 (至少 16 字节)
 *
 * 返回格式: "XXXXXNN" (5字符 + 2位十进制数字)
 */
void encode(const char *path, const char *time_str, const char *vd2, char *out)
{
    int i, len;
    uint32_t time_int, combined;
    char   *vd2_upper, *path_b64;
    unsigned char hmac_key[256];
    unsigned char hmac_out[20];
    uint32_t hash_words[5];
    uint32_t w15;
    int digit_count;

    /* ---- Step 1: 预处理 vd2 ---- */
    len = (int)strlen(vd2);
    vd2_upper = (char *)malloc(len + 1);
    digit_count = 0;

    for (i = 0; i < len; i++) {
        char ch = vd2[i];
        if (ch >= 'a' && ch <= 'z') {
            vd2_upper[i] = ch - 0x20;  /* 转大写 */
        } else {
            vd2_upper[i] = ch;
        }
        if (ch >= '0' && ch <= '9')
            digit_count++;
    }
    vd2_upper[len] = '\0';

    /* ---- Step 2: atoi(time) + digit_count → 全局状态 ---- */
    time_int = (uint32_t)atoi(time_str);
    combined = time_int + (uint32_t)digit_count;

    /* 4字节大端分解 (模拟 native 0x58c-0x598) */
    for (i = 0; i < 4; i++)
        g_keys[3 - i] = (combined >> (i * 8)) & 0xFF;

    /* ---- Step 3: Base64 编码 path ---- */
    path_b64 = (char *)malloc(((strlen(path) + 2) / 3) * 4 + 1);
    base64_encode((const unsigned char *)path, (int)strlen(path), path_b64);

    /* ---- Step 4: 构建 HMAC 密钥 ---- */
    /* key = HMAC_KEY + 4字节 time_key */
    int hklen = sizeof(HMAC_KEY) - 1;  /* 去掉末尾 '\0' */
    memcpy(hmac_key, HMAC_KEY, hklen);
    /* 追加4字节大端 key */
    for (i = 0; i < 4; i++)
        hmac_key[hklen + i] = (combined >> (24 - i * 8)) & 0xFF;
    int full_klen = hklen + 4;

    /* ---- Step 5: HMAC 计算 ---- */
    hmac_custom(hmac_key, full_klen,
                (const unsigned char *)path_b64, (int)strlen(path_b64),
                hmac_out);

    /* 解析前20字节为5个32位小端字 (HMAC输出) */
    for (i = 0; i < 5; i++)
        hash_words[i] = ((uint32_t)hmac_out[i * 4]) |
                        ((uint32_t)hmac_out[i * 4 + 1] << 8) |
                        ((uint32_t)hmac_out[i * 4 + 2] << 16) |
                        ((uint32_t)hmac_out[i * 4 + 3] << 24);

    /* ---- Step 6: Base58 编码 (连续除以58取余, 查表) ---- */
    /* 汇编: rev + &0x7FFFFFFF 取 HMAC 首字的字节反转值 */
    {
        uint32_t raw = hash_words[0];
        uint32_t rev;
        uint8_t rems[5];

        rev = ((raw & 0xFF) << 24) | ((raw & 0xFF00) << 8) |
              ((raw & 0xFF0000) >> 8) | ((raw >> 24) & 0xFF);
        w15 = rev & 0x7FFFFFFF;

        for (i = 0; i < 5; i++) {
            rems[i] = (uint8_t)(w15 % 58);
            out[i]  = CHARSET[rems[i] % 16];
            w15    /= 58;
        }

        /* ---- Step 7: AES MixColumns 扩散 → 纵向和 → mod 100 校验 ---- */
        /* 取后4个余数做 GF(2^8) MixColumns, 求和后模100 */
        uint32_t sum = mix_checksum(rems[1], rems[2], rems[3], rems[4]);
        uint32_t cs = sum % 100;

        out[5] = (char)('0' + cs / 10);
        out[6] = (char)('0' + cs % 10);
        out[7] = '\0';
    }

    free(vd2_upper);
    free(path_b64);
}

/* ============================================================
 * 测试
 * ============================================================ */

int main(void)
{
    printf("=== encode_native 测试 ===\n\n");

    /* 测试自定义哈希 */
    unsigned char hash_out[20];
    custom_hash((const unsigned char *)"hello", 5, hash_out);
    printf("[0] custom_hash(\"hello\") = ");
    for (int i = 0; i < 20; i++) printf("%02x", hash_out[i]);
    printf("\n");

    /* 测试 HMAC */
    unsigned char hmac_out[20];
    hmac_custom((const unsigned char *)"key", 3,
                (const unsigned char *)"message", 7, hmac_out);
    printf("[1] HMAC(\"key\", \"message\") = ");
    for (int i = 0; i < 20; i++) printf("%02x", hmac_out[i]);
    printf("\n");

    /* 测试 encode */
    char result[16];
    encode("/account/get_login_code/",
           "1760637928",
           "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d1",
           result);
    printf("[2] encode = \"%s\"\n", result);

    printf("\n=== 测试完成 ===\n");
    return 0;
}

```

接下来看getVA方法，在libhbsecurity.so 这个文件中，通过注册表看结构体，定位到sub_11B9E4中，这个函数非常复杂，下面是一些分析
```shell

  sub_11B9E4(context, a2, context, vd2)
  │
  ├─ [1] 初始化 BLAKE2b 状态
  │      sub_C3F3C(global_state + 168)
  │
  ├─ [2] 获取包名 → 喂入 BLAKE2b
  │      getPackageName() → sub_11972C()
  │
  ├─ [3] 获取签名信息
  │      getPackageManager().getPackageInfo(pkgName, 0)
  │      → packageInfo.signatures[0].hashCode()
  │
  ├─ [4] 签名校验 (证书固定/Certificate Pinning)
  │      hashCode → hex → 必须等于 "67780190"
  │      不匹配 → LABEL_324 → 返回空值
  │
  ├─ [5] 包名校验 (白名单)
  │      只允许 4 个包名之一:
  │      ├─ "com.dotamax.app"       (15字符)
  │      ├─ "com.max.xiaoheihe"     (17字符) ← 目标App
  │      ├─ "com.max.heyboxchat"    (18字符)
  │      └─ "com.max.maxaccelerator"(22字符)
  │      不匹配 → LABEL_324 → 返回空值
  │
  ├─ [6] BLAKE2b 混合计算
  │      sub_1199B4()  → 用包名生成内部状态
  │      sub_119630()  → 混合签名数据
  │      sub_119E8C()  → 混合 vd2 hashCode
  │      sub_16E43C()  → BLAKE2b 最终混元
  │      sub_135488()  → 转 hex 字符串
  │
  └─ [7] 构造 Java String 返回
         new String(hexBytes, "UTF-8") → 返回给 Java 层

  ---
  关键发现

  1. 签名硬编码校验 "67780190"

  if ( *v36 != 0x3039313038373736LL )  // "67780190"
      goto LABEL_324;  // 校验失败，返回空

  这是 signatures[0].hashCode() 的 hex 值。如果 APK
  被重新签名（二次打包），hashCode 必变，函数直接拒绝工
  作。这是经典的反篡改/证书固定机制。

  2. 包名白名单

  函数内硬编码了 4
  个合法包名，通过长度分发做字符串比较：

  ┌────────────────────────┬──────┐
  │          包名          │ 长度 │
  ├────────────────────────┼──────┤
  │ com.dotamax.app        │ 0xF  │
  ├────────────────────────┼──────┤
  │ com.max.xiaoheihe      │ 0x11 │
  ├────────────────────────┼──────┤
  │ com.max.heyboxchat     │ 0x12 │
  ├────────────────────────┼──────┤
  │ com.max.maxaccelerator │ 0x16 │
  └────────────────────────┴──────┘

  对不上任何一个也直接返回空。

  3. BLAKE2b 才是加密核心

  函数末尾的调用链才是真正的 hkey 生成：
  签名数据 + 包名 + 内部状态 + vd2_hashCode
      ↓ BLAKE2b 混合
      ↓ hex 编码
      ↓ new String(hex, "UTF-8")
      → hkey (8位十六进制字符串)

  而之前的 sub_11972C 虽然也叫 BLAKE2b，但在 getVA
  里它是用作 update 逐步喂入数据（包名、签名等），最后才
   final 产出哈希。

  ---
  总结

  getVA 做的事情比单纯加密多得多：

  4. 防二次打包 — 签名 hashCode 必须等于 0x67780190
  5. 防包名伪造 — 只认 4 个白名单包名
  6. 生成 hkey — 通过 BLAKE2b 混合 包名 + 签名 +
  内部状态 + vd2_hashCode → hex 字符串

  所以攻击者想要伪造
  hkey，必须先绕过签名校验和包名校验，不是单纯复现
  BLAKE2b 算法就够的。

```

后面的7字节nounce经过BLAKE2b哈希过后生成大概64字节，取最后4字节hex表示，就成了请求中的hkey
### frida验证


## 1.APK 基本信息

|项目|值|
|---|---|
|包名|`com.max.xiaoheihe`|
|版本|1.3.368|
|目标 SDK|33 (Android 13)|
|签名证书|CN=qf, SHA1=29:E0:8D:9B:EC:02:43:58:64:0B:C7:90:47:51:3F:26:78:41:CD:F6|

### Native Libraries

|文件|大小|用途|
|---|---|---|
|`libnative-lib.so`|~26KB|NDKTools JNI: encode, checkSignature, getrsakey|
|`libhbsecurity.so`|1.7MB|SecurityTool JNI: getVA, getVD, 13 个 native 方法, 静态链接 libsodium|

---

## 2. Java 调用链

### 2.1 核心入口 (p0.java:742-771)

HashMap map2 = new HashMap(16);  

// Step 1: 生成 seed  
String seed = SecurityTool.getVX(ctx, "PAENEHAMGACOBHIEMIHIJLKJPMMHJMMQABCNGBPPENCENP");  

// Step 2: 生成 vd2  
String vd2 = SecurityTool.getVD(ctx, seed, timeStr, userId);  
// → 32-char alphanumeric, 例: "kH11T1V1slsksIf0g0TeMu1g2IGn7dU7"  

// Step 3: 存储内部状态 (供 native getVA 后续读取)  
SecurityTool.setKN(timeStr, vd2);   // 存储 time 到全局  
SecurityTool.setKB(path, vd2);      // 存储 path 到容器 (key=vd2)  
SecurityTool.setKM(timeStr, vd2);   // 存储 MAC key 到容器 (key=vd2)  

// Step 4: 计算 _s (返回值被丢弃!)  
NDKTools.encode(ctx, path, timeStr, vd2);  
// → 7-char, 例: "207NR10"  

// Step 5: 计算 hkey  
String hkey = SecurityTool.getVA(ctx, vd2);  
// → 8-char hex (uppercase), 例: "B63AC4CD"  

// URL 参数映射  
map2.put("_time", timeStr);  // O() = "_time"  
map2.put("nonce", vd2);      // L() = "nonce"  
map2.put("hky", hkey);       // N() = "hky"

### 2.2 其他调用点

|类|路径|种子|
|---|---|---|
|p0.java|WebUtils|PAENEHAMGACOBHIEMIHIJLKJPMMHJMMQABCNGBPPENCENP|
|q0.java|WsManager|SDFOGUOWEHTBSYYEWQWADZGASEL|
|i.java|RequestInterceptorImpl|HPPDCEAENEHBFHPASRDCAMNHJLAAPF|

---

## 3. encode 算法 — ✅ 已完全破解

**位置**: `libnative-lib.so` → `Java_com_starlightc_ucropplus_network_temp_TempEncodeUtil_encode`

### 3.1 算法步骤

输入: path="/app/client/hot_fix/", time="1779776122", vd2="kH11T1V1slsksIf0g0TeMu1g2IGn7dU7"  
输出: _s="207NR10" (7 字符)

**Step 1 — vd2 预处理**: 小写转大写, 统计数字个数

vd2_upper = vd2.upper()           # "kH11T1V1..." → "KH11T1V1..."  
digit_count = count_digits(vd2)   # 例: 4

**Step 2 — 计算 combined**:

combined = atoi(time) + digit_count   # 1779776122 + 4 = 1779776126

**Step 3 — Base64 编码 path**:

path_b64 = base64(path.encode())      # "/app/client/hot_fix/" → "L2FwcC9jbGllbnQvaG90X2ZpeC8="

**Step 4 — HMAC-SHA1**:

key = path_b64  
msg = b'\x00\x00\x00\x00' + struct.pack('>I', combined & 0xFFFFFFFF)  
hmac_out = HMAC-SHA1(key=key, msg=msg)  # 20 bytes

**Step 5 — 提取 DWORD**:

nibble = hmac_out[19] & 0xF           # 最后一个字节的低4位  
dword = hmac_out[nibble:nibble+4]     # 从 HMAC 输出中取 4 字节 (little-endian)

**Step 6 — bswap32 + 符号位清零**:

rev = bswap32(dword)                   # 字节反转  
val = rev & 0x7FFFFFFF                # 去掉最高位

**Step 7 — Base58 编码** (5 位):

for i in range(5):  
    remainder = val % 58  
    chars[i] = charset[remainder]  
    val //= 58

**Step 8 — 动态 Charset**: 26 固定 + 32 动态

固定: "2345JKMNPQRT6789BCDFGHVWXY"  (26 chars, .rodata 0xB30 + 0xE90 + "XY")  
动态: vd2_upper[:32]                  (取 vd2 前 32 字符)  
完整: 58 字符

**Step 9 — MixColumns 校验码**:

checksum = mix_checksum_native([ord(chars[1]), ord(chars[2]), ord(chars[3]), ord(chars[4])])  
// sub_2CD0: 3 轮 SIMD GF(2^8) xtime (不同于标准 AES MixColumns)  
// 最后 vaddvq_s32 % 100 产生 2 位数字  

_s = chars[0..4] + f"{checksum:02d}"   # 7 字符

### 3.2 完整 Python 实现

def encode(path: str, time_str: str, vd2: str) -> str:  
    # Step 1  
    digit_count = sum(1 for c in vd2 if '0' <= c <= '9')  
    vd2_upper = vd2.upper()  

    # Step 2  
    combined = int(time_str) + digit_count  

    # Step 3-4  
    path_b64 = base64.b64encode(path.encode('utf-8'))  
    msg = b'\x00\x00\x00\x00' + struct.pack('>I', combined & 0xFFFFFFFF)  
    hmac_out = hmac.new(path_b64, msg, hashlib.sha1).digest()  

    # Step 5-6  
    nibble = hmac_out[19] & 0xF  
    dword = struct.unpack_from('<I', hmac_out, nibble)[0]  
    rev = struct.unpack('>I', struct.pack('<I', dword))[0]  
    val = rev & 0x7FFFFFFF  

    # Step 7-8  
    charset = "2345JKMNPQRT6789BCDFGHVWXY" + vd2_upper[:32]  
    chars = []  
    for _ in range(5):  
        chars.append(charset[val % 58])  
        val //= 58  

    # Step 9  
    cs = mix_checksum_native([ord(chars[i]) for i in range(1, 5)])  
    return ''.join(chars) + f"{cs:02d}"

### 3.3 mix_checksum_native 函数

输入: chars[1]~chars[4] 的 ASCII 值 (4 个整数)  
算法: sub_2CD0 — ARM64 NEON SIMD, 3 轮 GF(2^8) xtime  
      不可约多项式 0x1B, lane 配对特定排列  
      最终 vaddvq_s32 求和 % 100  
输出: 0-99 的 2 位数字  

已验证: 所有 4 个 Frida 捕获测试用例通过 ✓

### 3.4 验证数据

|path|time|vd2|_s|状态|
|---|---|---|---|---|
|/account/get_login_code/|1760637928|a1b2...c5d1|6B95412|✓|
|(其他3条)||||✓|

---

## 4. libhbsecurity.so 架构

### 4.1 JNI 方法注册

JNI_OnLoad 位于 `0x11D9C8`, 通过 `RegisterNatives` 动态注册 13 个方法。

JNINativeMethod 表在磁盘上被清零 (VMA 0x179038, file 0x178038), 但函数地址可从 `.rela.dyn` 的 `R_AARCH64_RELATIVE` 重定位提取:

Entry  0: setKA    (String)V                     → 0x11A708  
Entry  1: setKB    (String,String)V              → 0x11AD74  
Entry  2: setKM    (String,String)V              → 0x11AEDC  
Entry  3: setKT    (String,String)V              → 0x11B044  
Entry  4: setKN    (String,String)V              → 0x11B194  
Entry  5: setKD    (String,String)V              → 0x11B2E0  
Entry  6: setKC    (String,String)V              → 0x11B448  
Entry  7: getVX    (Context,String)String        → 0x11B7A4  
Entry  8: getVA    (Context,String)String        → 0x11B9E4  ← 主目标  
Entry  9: getVB    (int)int                      → 0x11D078  
Entry 10: getVC    (Context,String)String        → 0x11D114  
Entry 11: getVD    (Context,String,String,String)String → 0x11D334  
Entry 12: resetVA  ()V                           → 0x11D974

### 4.2 ELF 内存布局

LOAD (r-x): VMA 0x000000, file 0x000000, size 0x173450  — .text (代码)  
LOAD (rw-): VMA 0x174450, file 0x173450, size 0x58D0   — .data.rel.ro  
LOAD (rw-): VMA 0x17AD20, file 0x178D20, size 0x258E0  — .data + .bss  

文件偏移 = VMA - 段 p_vaddr + 段 p_offset  
对于 .text 段: file_offset = VMA (因为 p_vaddr=0, p_offset=0)

### 4.3 关键字符串

|地址 (VMA)|内容|用途|
|---|---|---|
|`0x7E5DD`|`000102030405...99`|2位十进制格式化表|
|`0x7E6B0`|`(Ljava/lang/String;I)Landroid/content/pm/PackageInfo;`|JNI 方法签名|
|`0x7EC8F`|`BLAKE2B_BLOCKBYTES`|BLAKE2b 常量名前缀|
|`0x7EE32`|`int _sodium_blake2b_final(...)`|libsodium 内部函数声明|
|`0x7F3B8`|`0123456789abcdef`|小写 hex 表|
|`0x7F3DC`|`int crypto_generichash_blake2b_final(...)`|BLAKE2b final 导出声明|
|`0x8070D`|`getPackageManager`|Android API 调用|
|`0x8071F`|`getPackageInfo`|Android API 调用|
|`0x80BEF`|`0123456789`|十进制数字表|
|`0x80CEE`|`signatures`|APK 签名字段名|
|`0x81407`|`java/lang/String`|JNI FindClass|
|`0x814C7`|`0123456789ABCDEF`|**getVA hex 编码表**|
|`0x814D8`|`getVA`|JNI 方法名|
|`0x81E09`|`/ABCDEFGHIJKLMN...0123456789+-_.=/:,@`|Base64/编码表|
|`0x81EE0`|BLAKE2b IV[0] = `6a09e667f3bcc908`|BLAKE2b 初始化向量|
|`0x8A0D0`|BLAKE2b IV[0] 第二副本|BLAKE2b 初始化向量|

### 4.4 libsodium 集成

libhbsecurity.so 静态链接 libsodium, 使用 `crypto_generichash_blake2b` 系列函数。

- **一次性初始化**: `0xC3F3C` — pthread_mutex 保护的 `sodium_init` call-once 模式
    
- **BLAKE2b IV**: 在 `.rodata` 中出现 2 次 (0x81EE0, 0x8A0D0)
    

---

## 5. getVA 分析 — ⚠️ 部分完成

### 5.1 函数入口 (0x11B9E4)

JNI 签名: getVA(Context, String)String  
寄存器: x0=JNIEnv*, x1=jclass, x2=Context, x3=vd2(String)  

大小: ~0x1730 bytes (至 0x11D114)  
特性: 控制流平坦化混淆

### 5.2 关键发现

**JNI 回调 Java API**: getVA 通过 JNI 回调获取 APK 签名:

1. `Context.getPackageManager()` — 获取 PackageManager
    
2. `PackageManager.getPackageInfo(packageName, GET_SIGNATURES)`
    
3. `PackageInfo.signatures[0]` — 取第一个签名
    
4. 可能的后续调用: `Signature.hashCode()` 或 `Signature.toByteArray()`
    

调用通过 JNI varargs 包装函数 `0x11972C` (CallObjectMethodV) 和 `0x11A014` (CallIntMethodV) 进行。

**BLAKE2b 调用**:

- 在 `0x11BC04` 处调用 `bl 0x11972C` 时传入 `w4=0x40` (64-byte output)
    
- 但实际 hkey 只有 8 hex chars = 4 bytes
    

**Hex 编码**:

- 使用表 `0123456789ABCDEF` @ 0x814C7
    
- 输出格式: `format(int_value, 'X')` — 大写, 无前导零
    

### 5.3 Setter 存储模型

**setKN (0x11B194)** — 直接存储到全局:

g_state + 0x00: 16 bytes string data (SSO buffer) — time 字符串  
g_state + 0x10: 数据指针 (heap overflow)

**setKB (0x11AD74)** — 存入 hash map:

g_state + 0xD0: std::unordered_map (或类似结构)  
  key = vd2 (std::string, 条目偏移 0x20)  
  value = path (std::string, 条目偏移 0x38)  
  条目大小: ~0x48 bytes

**setKM (0x11AEDC)** — 存入同一 hash map:

g_state + 0xD0: 同上容器  
  key = vd2 (条目偏移 0x20)  
  value = time (条目偏移 0x50)  
  条目大小: ~0x60 bytes

容器查找函数: `0x1194B4` — 遍历链表/hash bucket, 使用 memcmp 比较 key

### 5.4 Frida 捕获数据

|#|vd2|path|time|hkey|_s|
|---|---|---|---|---|---|
|1|kH11T1V1slsksIf0g0TeMu1g2IGn7dU7|/app/client/hot_fix/|1779776122|**B63AC4CD**|207NR10|
|2|ynVgfFrnIl07et8d8BlHdTgOAIueGd7N|/account/privacy/version/|1779776122|**3C1BFD49**|NYB4218|
|3|eO7dgMNmz2nummtHAmuUHI8819dI807B|/account/popup_v2/|1779776122|(未捕获)|IM2TV10|

### 5.5 待验证假设

hkey = 8 hex chars = 4 bytes → BLAKE2b(digest_size=4) 类似输出

可能公式 (均未匹配):

- ✗ H0-H3: 基础 BLAKE2b(vd2,4) / keyed(path) BLAKE2b
    
- ✗ H9-H14: keyed(sig_sha1/sig_sha256) + vd2/path/time 组合
    
- ✗ H15-H17: BLAKE2b(out=64) hex 编码
    
- ✗ HMAC-SHA1/SHA256(key=path) 截断 4 bytes
    
- ✗ HMAC-SHA1/SHA256(key=vd2, msg=path+time)
    

**需要进一步测试的方向**:

1. APK signature 的正确数据格式 (not SHA1 hex string, but `Signature.toByteArray()` or `Signature.hashCode()`)
    
2. BLAKE2b 的 key 可能由 setKM 存储的值派生
    
3. 输入可能有特殊分隔符或编码
    
4. 可能需要提取实际的 PackageInfo.signatures 字节数据
    

---

## 6. getVD 分析 — ⚠️ 未完成

### 6.1 函数入口 (0x11D334)

JNI 签名: getVD(Context, String, String, String)String  
输入: seed, timeStr, userId  
输出: 32-char alphanumeric (混合大小写)

### 6.2 已知特征

- 调用 `gmtime()` — GMT 时间转换
    
- 使用同一全局容器 `g_state + 0x78` (存储 time 字符串)
    
- 分配 `malloc(0x68)` — 104 bytes 新条目
    
- 遍历容器查找/插入由 seed 键控的条目
    
- BLAKE2b 参与计算
    

---

## 7. Frida Hook 脚本

### 7.1 hook_full_chain.js — 完整链路捕获

**设计目标**: Java 层全链路 + Native BLAKE2b 拦截 + 状态关联

- Java 层: getVX → getVD → setKN/setKB/setKM → encode → getVA
    
- Native 层: BLAKE2b IV 常量扫描 + `crypto_generichash_blake2b_final` export hook
    
- 状态关联: 使用 `pending` 对象将同一请求的所有 setter 调用关联到一起
    
- 自动保存: 每 30 秒保存至 `/sdcard/hkey_full_chain.json`，捕获 ≥5 条时立即保存
    

/**  
 * Frida hook — 完整 hkey 链路捕获 + BLAKE2b native 拦截  
 *  
 * 目标: 捕获 vd2→hkey 配对, 同时从 native 层确认 BLAKE2b 参数  
 *  
 * 用法:  
 *   frida -U -f com.max.xiaoheihe -l hook_full_chain.js  
 *   frida -U com.max.xiaoheihe -l hook_full_chain.js  
 *  
 * 输出: /sdcard/hkey_full_chain.json  
 */  

var CONSOLE_LOG = true;  
var captures = [];  
var savePath = "/sdcard/hkey_full_chain.json";  

function log(msg) { if (CONSOLE_LOG) console.log("[ENI] " + msg); }  

function saveNow() {  
    if (captures.length === 0) return;  
    try {  
        var json = JSON.stringify(captures, null, 2);  
        var File = Java.use("java.io.File");  
        var FileWriter = Java.use("java.io.FileWriter");  
        var f = File.$new(savePath);  
        var fw = FileWriter.$new(f);  
        fw.write(json);  
        fw.close();  
        log("Saved " + captures.length + " captures to " + savePath);  
    } catch (e) {  
        log("Save failed: " + e);  
        console.log(JSON.stringify(captures, null, 2));  
    }  
}  

// ============================================================  
// Part 1: Native BLAKE2b hooks in libhbsecurity.so  
// ============================================================  
function hookNativeBlake2b() {  
    var mod = Process.findModuleByName("libhbsecurity.so");  
    if (!mod) {  
        log("libhbsecurity.so not loaded yet, will retry...");  
        return false;  
    }  
    log("libhbsecurity.so @ " + mod.base + " size=" + mod.size);  

    // BLAKE2b IV[0] = 0x6a09e667f3bcc908 (little-endian)  
    var ivPattern = "08 C9 BC F3 67 E6 09 6A";  
    var ivMatches = Memory.scanSync(mod.base, mod.size, ivPattern);  

    if (ivMatches.length === 0) {  
        log("BLAKE2b IV not found by pattern scan");  
        return false;  
    }  

    log("Found BLAKE2b IV at " + ivMatches.length + " location(s)");  
    for (var i = 0; i < Math.min(ivMatches.length, 4); i++) {  
        log("  IV[" + i + "] @ " + ivMatches[i].address +  
            " (offset 0x" + ivMatches[i].address.sub(mod.base).toString(16) + ")");  
    }  

    // Hook crypto_generichash_blake2b_final if exported  
    var finalSym = Module.findExportByName("libhbsecurity.so", "crypto_generichash_blake2b_final");  
    if (!finalSym) {  
        // Try to find it by another name or pattern  
        var exports = mod.enumerateExports();  
        for (var j = 0; j < exports.length; j++) {  
            if (exports[j].name.indexOf("blake2b") !== -1) {  
                log("Found BLAKE2b export: " + exports[j].name + " @ " + exports[j].address);  
                finalSym = exports[j].address;  
            }  
        }  
    }  

    if (finalSym) {  
        log("Hooking crypto_generichash_blake2b_final @ " + finalSym);  
        Interceptor.attach(finalSym, {  
            onEnter: function (args) {  
                // args[0] = state, args[1] = output buffer, args[2] = output length  
                var outLen = args[2].toInt32();  
                this.outLen = outLen;  
                this.outBuf = args[1];  
                log("[BLAKE2b_final] output_len=" + outLen);  
            },  
            onLeave: function (retval) {  
                if (this.outBuf && this.outLen > 0 && this.outLen <= 64) {  
                    var hex = "";  
                    for (var k = 0; k < this.outLen; k++) {  
                        var b = this.outBuf.add(k).readU8();  
                        hex += ("0" + b.toString(16)).slice(-2);  
                    }  
                    log("[BLAKE2b_final] hash=" + hex + " (" + this.outLen + " bytes)");  
                }  
            }  
        });  
        log("BLAKE2b final hooked ✓");  
    } else {  
        log("BLAKE2b final not found as export (likely static/inlined)");  
    }  

    return true;  
}  

// ============================================================  
// Part 2: Java-level hooks — full chain  
// ============================================================  
function hookJavaLayer() {  
    Java.perform(function () {  
        log("Java ready, setting up hooks...");  

        var SecurityTool = Java.use("com.max.security.SecurityTool");  
        var NDKTools = Java.use("com.max.xiaoheihe.utils.NDKTools");  

        // State tracking — match calls from the same request  
        var pending = null;  

        function flushPending() {  
            if (!pending) return;  
            captures.push(JSON.parse(JSON.stringify(pending)));  
            console.log("\n=== Chain Capture #" + captures.length + " ===");  
            console.log(JSON.stringify(pending, null, 2));  
            pending = null;  
        }  

        // --- getVX: seed generation ---  
        try {  
            SecurityTool.getVX.overload('android.content.Context', 'java.lang.String')  
                .implementation = function (ctx, str) {  
                var result = this.getVX(ctx, str);  
                if (pending) flushPending();  
                pending = {};  
                pending.getVX_arg = String(str);  
                pending.getVX_result = String(result);  
                log("[getVX] arg=" + String(str).substring(0,30) + "... → " + String(result));  
                return result;  
            };  
            log("getVX hooked ✓");  
        } catch (e) { log("getVX: " + e); }  

        // --- getVD: vd2 generation ---  
        try {  
            SecurityTool.getVD.overload('android.content.Context', 'java.lang.String',  
                'java.lang.String', 'java.lang.String')  
                .implementation = function (ctx, seed, timeStr, userId) {  
                var result = this.getVD(ctx, seed, timeStr, userId);  
                if (!pending) pending = {};  
                pending.seed = String(seed);  
                pending.time_str = String(timeStr);  
                pending.user_id = String(userId);  
                pending.vd2 = String(result);  
                pending.vd2_len = String(result).length;  
                log("[getVD] time=" + timeStr + " vd2(" + pending.vd2_len + ")=" + result);  
                return result;  
            };  
            log("getVD hooked ✓");  
        } catch (e) { log("getVD: " + e); }  

        // --- setKN / setKB / setKM: internal state ---  
        try {  
            SecurityTool.setKN.overload('java.lang.String', 'java.lang.String')  
                .implementation = function (a, b) {  
                if (!pending) pending = {};  
                pending.setKN_arg1 = String(a);  
                pending.setKN_arg2 = String(b);  
                log("[setKN] " + String(a) + " | " + String(b).substring(0,20) + "...");  
                return this.setKN(a, b);  
            };  
            log("setKN hooked ✓");  
        } catch (e) { log("setKN: " + e); }  

        try {  
            SecurityTool.setKB.overload('java.lang.String', 'java.lang.String')  
                .implementation = function (a, b) {  
                if (!pending) pending = {};  
                pending.setKB_arg1 = String(a);  
                pending.setKB_arg2 = String(b);  
                log("[setKB] " + String(a) + " | " + String(b).substring(0,20) + "...");  
                return this.setKB(a, b);  
            };  
            log("setKB hooked ✓");  
        } catch (e) { log("setKB: " + e); }  

        try {  
            SecurityTool.setKM.overload('java.lang.String', 'java.lang.String')  
                .implementation = function (a, b) {  
                if (!pending) pending = {};  
                pending.setKM_arg1 = String(a);  
                pending.setKM_arg2 = String(b);  
                log("[setKM] " + String(a) + " | " + String(b).substring(0,20) + "...");  
                return this.setKM(a, b);  
            };  
            log("setKM hooked ✓");  
        } catch (e) { log("setKM: " + e); }  

        // --- NDKTools.encode: _s generation ---  
        try {  
            NDKTools.encode.overload('java.lang.Object', 'java.lang.String',  
                'java.lang.String', 'java.lang.String')  
                .implementation = function (ctx, path, timeStr, vd2) {  
                var result = this.encode(ctx, path, timeStr, vd2);  
                if (!pending) pending = {};  
                pending.path = String(path);  
                pending.time_str = pending.time_str || String(timeStr);  
                pending.vd2 = pending.vd2 || String(vd2);  
                pending._s = String(result);  
                log("[encode] path=" + String(path) + " → _s=" + String(result));  
                return result;  
            };  
            log("encode hooked ✓");  
        } catch (e) { log("encode: " + e); }  

        // --- getVA: hkey generation (THE TARGET) ---  
        try {  
            SecurityTool.getVA.overload('android.content.Context', 'java.lang.String')  
                .implementation = function (ctx, vd2) {  
                var result = this.getVA(ctx, vd2);  
                if (!pending) pending = {};  
                pending.vd2 = pending.vd2 || String(vd2);  
                pending.hkey = String(result);  
                pending.hkey_len = String(result).length;  
                pending.hkey_int = parseInt(String(result), 16);  
                log("[getVA] vd2=" + String(vd2) + " → hkey(" + pending.hkey_len + ")=" + String(result));  

                // getVA is the last call — flush  
                flushPending();  
                if (captures.length >= 5) saveNow();  
                return result;  
            };  
            log("getVA hooked ✓");  
        } catch (e) { log("getVA: " + e); }  

        // Periodic save  
        setInterval(saveNow, 30000);  
        log("All Java hooks ready. Trigger a network request...");  
    });  
}  

// ============================================================  
// Main  
// ============================================================  
log("Script loaded.");  

// Try native hooks immediately (library might already be loaded)  
setTimeout(function () {  
    hookNativeBlake2b();  
}, 500);  

// Also retry after a delay (library loads late)  
setTimeout(function () {  
    hookNativeBlake2b();  
}, 3000);  

// Java hooks  
hookJavaLayer();

### 7.2 hook_native_getva.js — Native 层直接拦截

**设计目标**: 在 getVA JNI 函数入口 (0x11B9E4) 直接 `Interceptor.attach`, 读取 jstring 参数和返回值

- 通过 `GetStringUTFChars` (JNI 函数表 offset 0xF8) 读取入参 vd2 和返回值 hkey
    
- 已知函数偏移 0x11B9E4 (来自 `.rela.dyn` 重定位表), 无需等 Java 层
    
- 双重保障: native hook + Java fallback hook
    
- 输出至 `/sdcard/native_getva_capture.json`
    

/**  
 * Frida hook — Native getVA + BLAKE2b 精确拦截  
 *  
 * 基于静态分析发现:  
 *   getVA     @ libhbsecurity.so + 0x11B9E4  
 *   getVD     @ libhbsecurity.so + 0x11D334  
 *   BLAKE2b IV @ 0x81EE0, 0x8A0D0 (in .rodata)  
 *   getVA 内部调用 BLAKE2b 时 outlen=64 (0x40)  
 *   Hex 编码表: "0123456789ABCDEF"  
 *  
 * 用法:  
 *   frida -U -f com.max.xiaoheihe -l hook_native_getva.js  
 *  
 * 输出: /sdcard/native_getva_capture.json  
 */  

var CONSOLE_LOG = true;  
var captures = [];  
var savePath = "/sdcard/native_getva_capture.json";  

function log(msg) { if (CONSOLE_LOG) console.log("[ENI] " + msg); }  

function saveNow() {  
    if (captures.length === 0) return;  
    try {  
        var json = JSON.stringify(captures, null, 2);  
        var File = Java.use("java.io.File");  
        var FileWriter = Java.use("java.io.FileWriter");  
        var f = File.$new(savePath);  
        var fw = FileWriter.$new(f);  
        fw.write(json);  
        fw.close();  
        log("Saved " + captures.length + " captures to " + savePath);  
    } catch (e) {  
        log("Save failed: " + e);  
        console.log(JSON.stringify(captures, null, 2));  
    }  
}  

// ============================================================  
// Part 1: Native BLAKE2b hooks via pattern scan  
// ============================================================  
var blake2bHooks = [];  
var blake2bStateMap = {}; // thread_id -> {key, keylen, outlen, input_data}  

function hookBlake2bFunctions() {  
    var mod = Process.findModuleByName("libhbsecurity.so");  
    if (!mod) {  
        log("libhbsecurity.so not loaded yet");  
        return false;  
    }  

    log("libhbsecurity.so @ " + mod.base + " size=0x" + mod.size.toString(16));  

    // BLAKE2b IV[0]: 0x6a09e667f3bcc908 → bytes 08 C9 BC F3 67 E6 09 6A  
    var ivPattern = "08 C9 BC F3 67 E6 09 6A";  
    var ivMatches = Memory.scanSync(mod.base, mod.size, ivPattern);  

    log("Found " + ivMatches.length + " BLAKE2b IV location(s)");  

    // For each IV location, find all xrefs from code  
    ivMatches.forEach(function (match, ivIdx) {  
        var ivAddr = match.address;  
        var ivOffset = ivAddr.sub(mod.base);  
        log("  IV[" + ivIdx + "] @ offset 0x" + ivOffset.toString(16));  
    });  

    return true;  
}  

// ============================================================  
// Part 2: Hook native getVA via known offset  
// ============================================================  
function hookNativeGetVA() {  
    var mod = Process.findModuleByName("libhbsecurity.so");  
    if (!mod) {  
        log("libhbsecurity.so not loaded yet");  
        return false;  
    }  

    // getVA JNI function at offset 0x11B9E4  
    var getVAAddr = mod.base.add(0x11B9E4);  
    log("Native getVA @ " + getVAAddr);  

    Interceptor.attach(getVAAddr, {  
        onEnter: function (args) {  
            // JNI function signature: getVA(JNIEnv* env, jclass cls, jobject ctx, jstring vd2)  
            this.env = args[0];  
            this.vd2 = args[3];  // 4th arg = vd2 (jstring)  
            this.sp = this.context.sp;  

            // Read vd2 string from JNI jstring  
            var env = this.env;  
            var getStringUTFChars = new NativeFunction(  
                env.add(0xf8).readPointer(),  // GetStringUTFChars @ 0xF8  
                'pointer', ['pointer', 'pointer', 'pointer']  
            );  

            try {  
                var isCopy = Memory.alloc(1);  
                var utfChars = getStringUTFChars(env, this.vd2, isCopy);  
                if (!utfChars.isNull()) {  
                    this.vd2Str = utfChars.readCString();  
                    log("[getVA] ENTER vd2=" + this.vd2Str);  
                }  
            } catch (e) {  
                log("[getVA] failed to read vd2: " + e);  
            }  
        },  
        onLeave: function (retval) {  
            // retval is a jstring  
            var env = this.env;  
            var getStringUTFChars = new NativeFunction(  
                env.add(0xf8).readPointer(),  
                'pointer', ['pointer', 'pointer', 'pointer']  
            );  

            try {  
                var isCopy = Memory.alloc(1);  
                var utfChars = getStringUTFChars(env, retval, isCopy);  
                if (!utfChars.isNull()) {  
                    var hkey = utfChars.readCString();  
                    log("[getVA] LEAVE hkey=" + hkey);  

                    if (this.vd2Str) {  
                        captures.push({  
                            hook: "native_getVA",  
                            time: new Date().toISOString(),  
                            vd2: this.vd2Str,  
                            hkey: hkey,  
                            hkey_len: hkey.length,  
                            hkey_int: parseInt(hkey, 16)  
                        });  
                        console.log("\n=== [Native getVA Capture #" + captures.length + "] ===");  
                        console.log(JSON.stringify(captures[captures.length - 1], null, 2));  
                    }  
                }  
            } catch (e) {  
                log("[getVA] failed to read hkey: " + e);  
            }  

            if (captures.length >= 5) saveNow();  
        }  
    });  

    log("Native getVA hooked ✓");  
    return true;  
}  

// ============================================================  
// Part 3: Java-level hooks (fallback + full chain)  
// ============================================================  
function hookJavaLayer() {  
    Java.perform(function () {  
        log("Java ready, setting up hooks...");  

        var SecurityTool = Java.use("com.max.security.SecurityTool");  
        var NDKTools = Java.use("com.max.xiaoheihe.utils.NDKTools");  

        // Chain tracker  
        var pending = {};  

        // getVA — main target  
        SecurityTool.getVA.overload('android.content.Context', 'java.lang.String')  
            .implementation = function (ctx, vd2) {  
            var result = this.getVA(ctx, vd2);  

            var vd2Str = String(vd2);  
            var hkeyStr = String(result);  

            var entry = {  
                hook: "java_getVA",  
                time: new Date().toISOString(),  
                vd2: vd2Str,  
                vd2_len: vd2Str.length,  
                hkey: hkeyStr,  
                hkey_len: hkeyStr.length,  
                hkey_int: parseInt(hkeyStr, 16)  
            };  

            captures.push(entry);  
            console.log("\n=== [Java getVA] ===");  
            console.log("  vd2(" + entry.vd2_len + "): " + vd2Str);  
            console.log("  hkey(" + entry.hkey_len + "): " + hkeyStr);  
            console.log("  hkey_int: " + entry.hkey_int);  

            if (captures.length >= 5) saveNow();  
            return result;  
        };  

        // getVD — vd2 generation  
        SecurityTool.getVD.overload('android.content.Context', 'java.lang.String',  
            'java.lang.String', 'java.lang.String')  
            .implementation = function (ctx, seed, timeStr, userId) {  
            var result = this.getVD(ctx, seed, timeStr, userId);  
            pending.vd2 = String(result);  
            pending.seed = String(seed);  
            pending.time_str = String(timeStr);  
            pending.user_id = String(userId);  
            log("[getVD] time=" + timeStr + " vd2=" + String(result));  
            return result;  
        };  

        // encode — _s generation  
        NDKTools.encode.overload('java.lang.Object', 'java.lang.String',  
            'java.lang.String', 'java.lang.String')  
            .implementation = function (ctx, path, timeStr, vd2) {  
            var result = this.encode(ctx, path, timeStr, vd2);  
            pending.path = String(path);  
            pending._s = String(result);  
            log("[encode] path=" + String(path) + " time=" + timeStr + " _s=" + String(result));  
            return result;  
        };  

        // setKN/setKB/setKM — internal state  
        try {  
            SecurityTool.setKN.overload('java.lang.String', 'java.lang.String')  
                .implementation = function (a, b) {  
                pending.setKN_a = String(a);  
                pending.setKN_b = String(b);  
                log("[setKN] " + String(a) + " | " + String(b).substring(0, 20) + "...");  
                return this.setKN(a, b);  
            };  
            SecurityTool.setKB.overload('java.lang.String', 'java.lang.String')  
                .implementation = function (a, b) {  
                pending.setKB_a = String(a);  
                pending.setKB_b = String(b);  
                log("[setKB] " + String(a) + " | " + String(b).substring(0, 20) + "...");  
                return this.setKB(a, b);  
            };  
            SecurityTool.setKM.overload('java.lang.String', 'java.lang.String')  
                .implementation = function (a, b) {  
                pending.setKM_a = String(a);  
                pending.setKM_b = String(b);  
                log("[setKM] " + String(a) + " | " + String(b).substring(0, 20) + "...");  
                return this.setKM(a, b);  
            };  
            log("setKN/setKB/setKM hooked ✓");  
        } catch (e) { log("setter hooks: " + e); }  

        setInterval(saveNow, 30000);  
        log("All Java hooks ready. Trigger network requests...");  
    });  
}  

// ============================================================  
// Main  
// ============================================================  
log("Script loaded.");  

// Try native hooks (library might not be loaded yet)  
setTimeout(function () {  
    var ok = hookNativeGetVA();  
    if (ok) hookBlake2bFunctions();  
}, 500);  

// Retry after delay  
setTimeout(function () {  
    var ok = hookNativeGetVA();  
    if (ok) hookBlake2bFunctions();  
}, 2000);  

setTimeout(function () {  
    hookNativeGetVA();  
    hookBlake2bFunctions();  
}, 5000);  

// Java hooks  
hookJavaLayer();

### 7.3 hook_getva_deep.js — 增强 Java Hook

**设计目标**: 轻量级快速捕获, 仅 Java 层 hook getVA + getVD + getVX

- 不依赖 native 偏移, 纯 Java hook
    
- 自动保存每 15 秒
    
- 输出 `/sdcard/getva_capture.json`
    

/**  
 * Frida hook — 深入 libhbsecurity.so 捕获 getVA + BLAKE2b 内部调用  
 *  
 * 用法:  
 *   frida -U -f com.max.xiaoheihe -l hook_getva_deep.js  
 */  
var CONSOLE_LOG = true;  

function log(msg) {  
    if (CONSOLE_LOG) console.log("[ENI] " + msg);  
}  

// Find libhbsecurity.so module  
var hbsec = null;  
function getHbsec() {  
    if (hbsec) return hbsec;  
    var mods = Process.enumerateModules();  
    for (var i = 0; i < mods.length; i++) {  
        if (mods[i].name.indexOf("libhbsecurity") !== -1) {  
            hbsec = mods[i];  
            log("Found " + mods[i].name + " at " + mods[i].base);  
            return hbsec;  
        }  
    }  
    return null;  
}  

// Capture entries  
var captures = [];  

function saveNow() {  
    if (captures.length === 0) return;  
    try {  
        var json = JSON.stringify(captures, null, 2);  
        var File = Java.use("java.io.File");  
        var FileWriter = Java.use("java.io.FileWriter");  
        var f = File.$new("/sdcard/getva_capture.json");  
        var fw = FileWriter.$new(f);  
        fw.write(json);  
        fw.close();  
        log("Saved " + captures.length + " captures");  
    } catch (e) {  
        log("Save failed: " + e);  
        console.log(JSON.stringify(captures, null, 2));  
    }  
}  

// Hook BLAKE2b functions when libhbsecurity.so loads  
function hookBlake2b() {  
    var mod = getHbsec();  
    if (!mod) {  
        log("libhbsecurity.so not loaded yet, waiting...");  
        return false;  
    }  

    // Search for BLAKE2b IV[0] = 0x6a09e667f3bcc908  
    Module.enumerateRanges('r--').forEach(function(range) {  
        if (range.base.compare(mod.base) >= 0 &&  
            range.base.compare(mod.base.add(mod.size)) < 0) {  
            var matches = Memory.scanSync(range.base, range.size, "08 C9 BC F3 67 E6 09 6A");  
            if (matches.length > 0) {  
                log("Found BLAKE2b IV at " + matches[0].address);  
            }  
        }  
    });  

    return true;  
}  

Java.perform(function () {  
    log("Java ready, setting up hooks...");  

    // Hook SecurityTool.getVA  
    try {  
        var SecurityTool = Java.use("com.max.security.SecurityTool");  

        SecurityTool.getVA.overload('android.content.Context', 'java.lang.String')  
            .implementation = function (ctx, vd2) {  
            var result = this.getVA(ctx, vd2);  
            var vd2Str = String(vd2);  
            var hkey = String(result);  

            var entry = {  
                hook: "getVA",  
                time: new Date().toISOString(),  
                vd2: vd2Str,  
                vd2_len: vd2Str.length,  
                hkey: hkey,  
                hkey_len: hkey.length,  
                hkey_int: parseInt(hkey, 16),  
            };  

            captures.push(entry);  
            console.log("\n=== [getVA] ===");  
            console.log("  vd2 (" + vd2Str.length + "): " + vd2Str);  
            console.log("  hkey (" + hkey.length + "): " + hkey);  
            console.log("  hkey as int: " + entry.hkey_int);  
            console.log("================");  

            if (captures.length >= 5) saveNow();  
            return result;  
        };  
        log("getVA hooked ✓");  
    } catch (e) { log("getVA hook failed: " + e); }  

    // Hook SecurityTool.getVD  
    try {  
        var SecurityTool = Java.use("com.max.security.SecurityTool");  

        SecurityTool.getVD.overload('android.content.Context', 'java.lang.String',  
                                     'java.lang.String', 'java.lang.String')  
            .implementation = function (ctx, seed, timeStr, userId) {  
            var result = this.getVD(ctx, seed, timeStr, userId);  
            var entry = {  
                hook: "getVD",  
                time: new Date().toISOString(),  
                seed: String(seed),  
                time_str: String(timeStr),  
                user_id: String(userId),  
                vd2: String(result),  
                vd2_len: String(result).length,  
            };  
            captures.push(entry);  
            console.log("\n=== [getVD] ===");  
            console.log("  seed: " + entry.seed.substring(0, 40) + "...");  
            console.log("  time: " + entry.time_str);  
            console.log("  uid:  " + entry.user_id);  
            console.log("  vd2 (" + entry.vd2_len + "): " + entry.vd2);  
            console.log("================");  
            return result;  
        };  
        log("getVD hooked ✓");  
    } catch (e) { log("getVD hook failed: " + e); }  

    // Hook SecurityTool.getVX  
    try {  
        var SecurityTool = Java.use("com.max.security.SecurityTool");  
        SecurityTool.getVX.overload('android.content.Context', 'java.lang.String')  
            .implementation = function (ctx, str) {  
            var result = this.getVX(ctx, str);  
            console.log("\n=== [getVX] ===");  
            console.log("  input: " + String(str));  
            console.log("  seed:  " + String(result));  
            console.log("================");  
            return result;  
        };  
        log("getVX hooked ✓");  
    } catch (e) { log("getVX hook failed: " + e); }  

    // Periodic save  
    setInterval(saveNow, 15000);  

    log("All hooks ready. Trigger network requests...");  
});  

// Also try to hook native BLAKE2b when lib loads  
setTimeout(function () {  
    hookBlake2b();  
}, 2000);  

log("Script loaded.");

---

## 8. Frida 验证实验 — 实际运行过程

### 8.1 环境准备

设备: M2012K10C (Xiaomi), Android arm64-v8a  
Frida: 17.2.12 (PC 端 & 设备端版本一致)  
PC: frida --version → 17.2.12  
设备: /data/local/tmp/frida-server-17.2.12-android-arm64  
App: com.max.xiaoheihe v1.3.368 (已安装)  
Root: 已获取

连接测试:

$ frida-ps -U | head -5  
  PID  Name  
-----  -----  
18598  Alpha  
 6155  MI_RIC  
17982  QQ  

 OK, Frida 正常工作

### 8.2 启动 App + 注入 Hook

# 清理旧捕获文件  
$ adb shell "rm -f /sdcard/hkey_full_chain.json /sdcard/native_getva_capture.json"  

# 停止 App (确保冷启动)  
$ adb shell "am force-stop com.max.xiaoheihe"  

# 启动 App 并注入 hook_full_chain.js  
$ frida -U -f com.max.xiaoheihe -l hook_full_chain.js

注入过程输出:

____  
    / _  |   Frida 17.2.12 - A world-class dynamic instrumentation toolkit  
   | (_| |  
    > _  |   Commands:  
   /_/ |_|       help      -> Displays the help system  
   . . . .       object?   -> Display information about 'object'  
   . . . .       exit/quit -> Exit  
   . . . .  
   . . . .   Connected to M2012K10C (id=jreatg89irtoyxxs)  
Spawning `com.max.xiaoheihe`...  
[ENI] Script loaded.  
Spawned `com.max.xiaoheihe`. Resuming main thread!

### 8.3 Hook 初始化

App 启动后, Java 层 hook 立即生效:

[ENI] libhbsecurity.so not loaded yet, will retry...  
[ENI] Java ready, setting up hooks...  
[ENI] getVX hooked ✓  
[ENI] getVD hooked ✓  
[ENI] setKN hooked ✓  
[ENI] setKB hooked ✓  
[ENI] setKM hooked ✓  
[ENI] encode hooked ✓  
[ENI] getVA hooked ✓  
[ENI] All Java hooks ready. Trigger a network request...

libhbsecurity.so 延迟加载, setTimeout 重试成功:

[ENI] libhbsecurity.so @ 0x7d5b044000 size=1777664  
[ENI] Found BLAKE2b IV at 2 location(s)  
[ENI]   IV[0] @ 0x7d5b0c5ee0 (offset 0x81ee0)  
[ENI]   IV[1] @ 0x7d5b0ce0d0 (offset 0x8a0d0)

Native BLAKE2b final hook 失败 (函数未导出, 静态链接):

TypeError: not a function  
    at hookNativeBlake2b (hook_full_chain.js:63)

→ 结论: `crypto_generichash_blake2b_final` 未作为动态符号导出, 无法通过 `Module.findExportByName` 找到。需要改用 pattern scan 在代码段找调用点。

### 8.4 App 自动触发请求捕获

App 冷启动后自动发起了多组 API 请求, 以下为控制台实时输出:

**请求 #1**:

[ENI] [getVX] arg=HPPDCEAENEHBFHPASRDCAMNHJLAAPF... → NBsjPuiPSjRhlKrIA26jK2fOKlg4Wwqs  

=== Chain Capture #1 ===  
{  
  "getVX_arg": "HPPDCEAENEHBFHPASRDCAMNHJLAAPF",  
  "getVX_result": "NBsjPuiPSjRhlKrIA26jK2fOKlg4Wwqs"  
}

→ 此请求未完成全链路 (可能为预加载或缓存命中)

**请求 #2 — 完整链路**:

[ENI] [getVX] arg=HPPDCEAENEHBFHPASRDCAMNHJLAAPF... → BHsvQWtvFW1N28N79Cr7XCY6VTvjTLIb  
[ENI] [getVD] time=1779776122 vd2(32)=kH11T1V1slsksIf0g0TeMu1g2IGn7dU7  
[ENI] [setKN] 1779776122 | kH11T1V1slsksIf0g0Te...  
[ENI] [setKB] /app/client/hot_fix/ | kH11T1V1slsksIf0g0Te...  
[ENI] [setKM] 1779776122 | kH11T1V1slsksIf0g0Te...  
[ENI] [encode] path=/app/client/hot_fix/ → _s=207NR10  
[ENI] [getVA] vd2=kH11T1V1slsksIf0g0TeMu1g2IGn7dU7 → hkey(8)=B63AC4CD  

=== Chain Capture #3 ===  
{  
  "getVX_arg": "HPPDCEAENEHBFHPASRDCAMNHJLAAPF",  
  "getVX_result": "BHsvQWtvFW1N28N79Cr7XCY6VTvjTLIb",  
  "seed": "BHsvQWtvFW1N28N79Cr7XCY6VTvjTLIb",  
  "time_str": "1779776122",  
  "user_id": "null",  
  "vd2": "kH11T1V1slsksIf0g0TeMu1g2IGn7dU7",  
  "vd2_len": 32,  
  "setKN_arg1": "1779776122",  
  "setKN_arg2": "kH11T1V1slsksIf0g0TeMu1g2IGn7dU7",  
  "setKB_arg1": "/app/client/hot_fix/",  
  "setKB_arg2": "kH11T1V1slsksIf0g0TeMu1g2IGn7dU7",  
  "setKM_arg1": "1779776122",  
  "setKM_arg2": "kH11T1V1slsksIf0g0TeMu1g2IGn7dU7",  
  "path": "/app/client/hot_fix/",  
  "_s": "207NR10",  
  "hkey": "B63AC4CD",  
  "hkey_len": 8,  
  "hkey_int": 3057304781  
}

**请求 #3 — 第二条完整链路**:

[ENI] [getVX] arg=HPPDCEAENEHBFHPASRDCAMNHJLAAPF... → ytrAmHslijLtGbELQtpBB1QgGJwXXVVa  
[ENI] [getVD] time=1779776122 vd2(32)=ynVgfFrnIl07et8d8BlHdTgOAIueGd7N  
[ENI] [setKN] 1779776122 | ynVgfFrnIl07et8d8BlH...  
[ENI] [setKB] /account/privacy/version/ | ynVgfFrnIl07et8d8BlH...  
[ENI] [setKM] 1779776122 | ynVgfFrnIl07et8d8BlH...  
[ENI] [encode] path=/account/privacy/version/ → _s=NYB4218  
[ENI] [getVA] vd2=ynVgfFrnIl07et8d8BlHdTgOAIueGd7N → hkey(8)=3C1BFD49  

=== Chain Capture #4 ===  
{  
  "getVX_arg": "HPPDCEAENEHBFHPASRDCAMNHJLAAPF",  
  "getVX_result": "ytrAmHslijLtGbELQtpBB1QgGJwXXVVa",  
  "seed": "ytrAmHslijLtGbELQtpBB1QgGJwXXVVa",  
  "time_str": "1779776122",  
  "user_id": "null",  
  "vd2": "ynVgfFrnIl07et8d8BlHdTgOAIueGd7N",  
  "vd2_len": 32,  
  "setKB_arg1": "/account/privacy/version/",  
  "setKB_arg2": "ynVgfFrnIl07et8d8BlHdTgOAIueGd7N",  
  "path": "/account/privacy/version/",  
  "_s": "NYB4218",  
  "hkey": "3C1BFD49",  
  "hkey_len": 8,  
  "hkey_int": 1008565577  
}

### 8.5 捕获数据汇总

|#|vd2|path|time|_s|hkey|hkey_len|
|---|---|---|---|---|---|---|
|1|eO7dgMNmz2nummtHAmuUHI8819dI807B|/account/popup_v2/|1779776122|IM2TV10|(未触发)|—|
|2|kH11T1V1slsksIf0g0TeMu1g2IGn7dU7|/app/client/hot_fix/|1779776122|207NR10|**B63AC4CD**|8|
|3|ynVgfFrnIl07et8d8BlHdTgOAIueGd7N|/account/privacy/version/|1779776122|NYB4218|**3C1BFD49**|8|

关键发现:

- **hkey 固定为 8 hex chars (uppercase)** = 4 bytes → 排除了出长度=64 的 BLAKE2b 直接 hex 编码假设
    
- **vd2 固定为 32 chars** (混合大小写 alphanumeric)
    
- **_s 固定为 7 chars** (5 字符 + 2 位数字校验码)
    
- **所有请求使用相同 timestamp** (1779776122 = App 启动时一次性获取)
    
- **seed 每次请求不同** (getVX 返回值变化)
    

### 8.6 encode 交叉验证

用捕获到的 `_s` 值验证之前的 encode 实现:

$ python generate_hkey_v2.py --test

编码验证通过:

[1] encode 测试  
    path: /account/get_login_code/  
    time: 1760637928  
    vd2:  a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d1  
    → _s = '6B95412'     # 与 Frida 捕获一致 ✓

捕获数据验证 (`captured_data.json`):

$ python -c "  
from generate_hkey_v2 import encode  
# 捕获 #1  
s1 = encode('/app/client/hot_fix/', '1779776122', 'kH11T1V1slsksIf0g0TeMu1g2IGn7dU7')  
print(f'_s={s1} (expected 207NR10) {\"✓\" if s1==\"207NR10\" else \"✗\"}')  

# 捕获 #2  
s2 = encode('/account/privacy/version/', '1779776122', 'ynVgfFrnIl07et8d8BlHdTgOAIueGd7N')  
print(f'_s={s2} (expected NYB4218) {\"✓\" if s2==\"NYB4218\" else \"✗\"}')  
"

_s=207NR10 (expected 207NR10) ✓  
_s=NYB4218 (expected NYB4218) ✓

→ **encode 算法完全确认, 两个新鲜捕获全部通过!**

### 8.7 getVA 假设验证 (失败)

用捕获到的 `vd2→hkey` 配对测试全部 17 个假设:

$ python generate_hkey_v2.py --verify captured_data.json

加载 2 条捕获记录  

--- 记录 #1 ---  
  path: /app/client/hot_fix/  
  time: 1779776122  
  vd2:  kH11T1V1slsksIf0g0TeMu1g2IGn7dU7  
  hkey (actual): B63AC4CD  
  _s:   207NR10  

      H0: BLAKE2b(vd2,4): D0CA4682  
      H1: BLAKE2b(vd2,8): 5216EDCC9815EEF7  
      H2: keyed(path) BLAKE2b(vd2,4): 2994C95D  
      ...  
      H17: keyed(path+sig) BLAKE2b(time+vd2,64) hex: C8B0A117...  

  >>> 无假设匹配! 需要新的假设.  

--- 记录 #2 ---  
  path: /account/privacy/version/  
  time: 1779776122  
  vd2:  ynVgfFrnIl07et8d8BlHdTgOAIueGd7N  
  hkey (actual): 3C1BFD49  
  _s:   NYB4218  

      H0: BLAKE2b(vd2,4): D589CA75  
      ...  

  >>> 无假设匹配! 需要新的假设.

**全部 17 个假设均未匹配!** 详细测试矩阵:

|假设|R1 hkey (expected B63AC4CD)|R2 hkey (expected 3C1BFD49)|匹配?|
|---|---|---|---|
|H0: BLAKE2b(vd2,4)|D0CA4682|D589CA75|✗|
|H1: BLAKE2b(vd2,8)|5216EDCC9815EEF7|645E466A6136C941|✗|
|H2: keyed(path) BLAKE2b(vd2,4)|2994C95D|F0AEF7F0|✗|
|H3: keyed(path) BLAKE2b(time+vd2,4)|58D3A759|4C44F2F3|✗|
|H9: keyed(sig_sha1) BLAKE2b(vd2+path+time,8)|C819E4FACA5466CC|E3BBA11715D11B19|✗|
|H10: keyed(sig_sha1) BLAKE2b(path+time+vd2,8)|5B3DFEF18AB5AF37|9426B9A214757B2D|✗|
|H12: BLAKE2b(vd2+sig_hex,8)|CA863579191B18F1|7324C81D89538C7D|✗|
|H13: keyed(path) BLAKE2b(vd2+sig_sha1,8)|932081550C29DD2|60E81AD9E77E11C3|✗|
|H15: BLAKE2b(vd2+path+time,64) hex|CE4B1C81...|F4DA5161...|✗|
|H16: keyed(sig_sha1) BLAKE2b(vd2,64) hex|EFA6142A...|1CE1BF1C...|✗|
|H17: keyed(path+sig) BLAKE2b(time+vd2,64) hex|C8B0A117...|17C11EFE...|✗|

### 8.8 额外尝试: HMAC & 其他哈希

$ python3 -c "  
import hashlib, hmac  
# 验证 HMAC-SHA1(key=path) trunc4, HMAC-SHA256, 等  
# ... 测试了 ~30 种 HMAC/BLAKE2b 参数组合  
"

结果: **全部不匹配**

### 8.9 验证结论与后续方向

**确认的事实**:

1. ✅ `encode()` 算法完全正确 — 两个新鲜捕获均通过交叉验证
    
2. ✅ `hkey = 8 hex chars = 4 bytes` — 来自 BLAKE2b 或类似哈希的 int 格式化
    
3. ✅ `vd2 = 32 chars alphanumeric` — BLAKE2b + 特殊编码
    
4. ✅ `_s` 值被 `encode()` 正确计算但在 Java 层被丢弃
    
5. ✗ 全部 17 个 BLAKE2b 假设都不正确
    

**根本原因分析**:

- APK 签名字节格式错误 — 我们使用了 `SHA1: 29:E0:8D:...` 的 hex 字符串, 而 `PackageInfo.signatures[0]` 返回的 `Signature` 对象包含完整的 X.509 证书 DER 编码
    
- BLAKE2b key 来源未确认 — 可能是 `Signature.toByteArray()` (DER 编码字节), 或 `Signature.hashCode()` (Java int hashCode)
    
- 输入可能包含 app package name 或其他未追踪的参数
    
- `setKM(time, vd2)` 存储的 "MAC key" 可能参与 BLAKE2b key 派生
    

**下一步**:

1. Hook `PackageInfo.signatures[0].toByteArray()` 获取实际签名字节
    
2. Hook BLAKE2b init/update 函数 (pattern scan 代码段) 捕获中间状态
    
3. 测试签名字节作为 key 的 BLAKE2b 组合
    

---

## 9. 工具脚本

### 9.1 generate_hkey_v2.py

核心 Python 实现:

- `encode(path, time, vd2)` → `_s` ✅ 已完全验证
    
- `get_vd(time, seed, user_id)` → `vd2` ⚠️ stub (BLAKE2b hex, 实际格式不同)
    
- `get_va(vd2, path, time)` → `hkey` ⚠️ stub
    
- `get_va_hypotheses()` → 17 个假设生成
    
- `verify_captures(json_path)` → 自动验证捕获数据
    
- `compute(path, time)` → 一键计算
    

python generate_hkey_v2.py "/some/api/path/"           # 计算  
python generate_hkey_v2.py --test                        # 运行测试  
python generate_hkey_v2.py --verify captured_data.json   # 验证捕获

### 9.2 mix_checksum.py

sub_2CD0 MixColumns 校验码的独立实现:

def mix_checksum_native(bytes_1_to_4):  
    # 3 轮 GF(2^8) xtime SIMD 模拟  
    # 输入: chars[1]~chars[4] 的 ASCII 值  
    # 输出: 0-99

---

## 10. 关键 ELF 地址速查

|函数/数据|VMA|文件偏移|大小|
|---|---|---|---|
|JNI_OnLoad|0x11D9C8|0x11D9C8|0xBC|
|getVA|0x11B9E4|0x11B9E4|~0x1730|
|getVD|0x11D334|0x11D334|~0x640|
|setKN|0x11B194|0x11B194|0x14C|
|setKB|0x11AD74|0x11AD74|0x168|
|setKM|0x11AEDC|0x11AEDC|0x168|
|getVX|0x11B7A4|0x11B7A4|0x240|
|BLAKE2b wrapper|0x11972C|0x11972C|0x9C|
|JNI varargs wrapper|0x11A014|0x11A014|0x94|
|sodium_init call-once|0xC3F3C|0xC3F3C|—|
|hash map lookup|0x1194B4|0x1194B4|~0x80|
|hex formatter|0x11A0FC|0x11A0FC|—|
|BLAKE2b IV[0]|0x81EE0|0x81EE0|8 bytes|
|BLAKE2b IV[0] 副本|0x8A0D0|0x8A0D0|8 bytes|
|decimal table (00-99)|0x7E5DD|0x7E5DD|200 bytes|
|hex table (ABCDEF)|0x814C7|0x814C7|17 bytes|
|JNINativeMethod table|0x179038|0x178038|312 bytes (运行时解密)|

---

## 11. 当前状态总结

|组件|状态|完成度|
|---|---|---|
|encode (_s)|✅ 完全破解并验证|100%|
|mix_checksum|✅ 完全破解并验证|100%|
|JNI 架构映射|✅ 完全完成|100%|
|getVA (hkey)|⚠️ 部分完成|~60%|
|getVD (vd2)|⚠️ 初步分析|~30%|
|_s 传输方式|❌ 未知|0%|

{% endraw %}

# [如何获取 keep 跑步数据](https://github.com/yihong0618/gitblog/issues/191)

这些天为了把自己的 [running_page](https://github.com/yihong0618/running_page) 支持 keep，很大一部分精力都在和 keep 数据打交道，也有了一些心得，在这儿记录一下。没准过段儿时间我就成安全工程师了~
具体代码见 https://github.com/yihong0618/running_page/blob/master/scripts/keep_sync.py

## 如何获取到接口
这也算是比较重要的一步。我一般采取以下步骤
1. 去官网看看，有登陆的地方没，如果有获取到最原始的接口。比如 `api.gotokeep.com`
2. 如果没有的话，我会在 GitHub 上进行搜索，但是 keep 这个遇到了困难，叫 keep 的太多了。如果有人做过会节约非常多的时间，但不幸的是，keep 不像 `garmin, strava` 那样，几乎没人做过。keep, 悦跑圈，咕咚那样的国内软件也鲜有人做过。幸运的是我发现了有人 2 年多以前做了 [keepForMac](https://github.com/wodewone/keepForMac), 那时候 keep 的接口好像还是开放的。虽然这个项目已经不能用了，但我找到了登陆接口。
![image](https://user-images.githubusercontent.com/15976103/96837336-d5c4af00-1478-11eb-89d1-7219bda8407a.png)
![image](https://user-images.githubusercontent.com/15976103/96832936-2be22400-1472-11eb-81ee-94f148291c60.png)
3. 有登陆接口后我继续在 GitHub 搜索有没有人做过 keep 的模拟登陆，这次没有那么幸运了，做过登陆的只有这一个项目。
4. 没办法，只好抓包，我用 Charles 抓出跑步 logs 的接口，再碰碰运气，贴到 GitHub 中，一无所获，贴到 google 中，发现了[这篇文章](https://www.kai666666.top/2020/07/14/Keep-APP%E6%8A%80%E6%9C%AF%E7%A0%94%E7%A9%B6/) 嗯，证明我拿 keep 数据的想法是可行的。挺好。
## 搞定数据

1. 想拿到跑步数据需要，速度，时间，距离，心率。。。等等，除了这些还有生成可视化地图比较重要的经纬度，如果要生成 gpx 还需要时间戳。
2. 一般 经纬度 + 速度 + 时间 + 海拔的数据都是加密的，我之前搞过 [Runtastic](https://github.com/yihong0618/Runtastic) 是 base64 加密，然后一个一个猜最后搞定，如下图：
![image](https://user-images.githubusercontent.com/15976103/96838254-3bfe0180-147a-11eb-9ae5-02d461f74d7e.png)
3. 但同样的方式我去搞 keep 直接失败，甚至一个字节一个字节来碰运气发现还是不行。
4. 搞不定之后我尝试用之前看到的一个项目 [Ciphey](https://github.com/Ciphey/Ciphey) 尝试破解，不过只告诉我是 base 加密但解不出内容
5. 最后惊奇的发现好像好多内容（能访问的 txt 数据都是）都包括这个开头 `H4sIAAAAAAAA`, 本着我的凡事先搜索精神我把这段贴在了 GitHub 中，发现有好多代码出现了这个，肯定是某种加密。
![image](https://user-images.githubusercontent.com/15976103/96839102-4cfb4280-147b-11eb-8b39-26bcbf9e2c94.png)
![image](https://user-images.githubusercontent.com/15976103/96839186-69977a80-147b-11eb-88f0-a23e6fb173c8.png)
6. 在根据别人写的代码反推加密，就搞定了 (zip + base64)
![image](https://user-images.githubusercontent.com/15976103/96839309-977cbf00-147b-11eb-8629-274a263ff661.png)
7. 搞定了这个数据，就剩解析成我需要的格式，生成 polyline 了，存入数据库。这个我搞过好多，也就简单了。
## 总结
1. keep 的数据保护并不复杂，但是 keep 因为想把用户留住的原因省去了好多数据
2. 对于破译保护一定要多观察，冷静。站在前人的肩膀固然好，更要有自己的能力
3. 这些数据里 keep 粒度是最不好的，导出 gpx 会缺失很多，但悲凉的是已经是国产里最好的了

P.S. 本质是 base64 不算加密，做数据编码一定要记住。

---

@chendongcse 

谢谢，因为我数据较少，没有经过数据量的测试，你感兴趣能帮我试试有什么问题么？

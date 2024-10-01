
## 什么是骨架屏


在客户端开发中，我们总是需要等待拿到服务端的响应后，再将内容呈现到页面上，那么在用户发起请求到客户端成功拿到响应的这段时间内，应该在屏幕上呈现点什么好呢？


答案是：**骨架屏**


那么什么是骨架屏呢，来问下 GPT：



> 骨架屏（Skeleton Screen）是一种现代的用户界面设计技术，用于提升应用或网站在加载内容时的用户体验。在内容的完全加载和呈现之前，骨架屏提供了一种模糊的预览，形似最终内容的空白版，通常用灰色的块、线条或元素占位符表示。这种设计方法可以有效减少用户的感知等待时间，增强用户的交互感。
> 
> 
> ### 功能和用途
> 
> 
> 1. **提高感知性能**：骨架屏通过立即显示页面的基本结构（而非旋转的加载图标或完全空白的屏幕），给用户一种内容即将呈现的感觉，这可以使等待时间感觉上更短。
> 2. **改善用户体验**：使用骨架屏可以减少用户在加载过程中的焦虑，用户看到界面元素已经在逐步加载，会有更多的耐心等待最终内容的呈现。
> 3. **提供内容加载的视觉提示**：骨架屏体现了页面内容加载的进度，可以让用户知道哪些内容即将出现，这样用户就不会感到突然或困惑。
> 
> 
> ### 实现方式
> 
> 
> 骨架屏的实现通常包括以下几个步骤：
> 
> 
> 1. **设计**：设计与最终内容布局相似的基本框架，使用灰色或浅色块代表将要加载的各种元素，如文本行、图片、按钮等。
> 2. **前端实现**：在前端代码中，可以使用HTML和CSS来创建这些占位符。对于复杂的动态加载内容，可以使用JavaScript或前端框架如React、Vue等来动态控制骨架屏的显示和隐藏。
> 3. **数据加载后的处理**：一旦相应的数据加载完成，骨架屏应被实际内容替换。这通常涉及到监听数据加载的完成事件，然后更新UI。
> 
> 
> ### 示例
> 
> 
> 在一个简单的网页应用中，如果你正在加载一个包含标题、几段文本和图片的文章，骨架屏可能包括：
> 
> 
> * 一个灰色的矩形块预留给图片。
> * 几个灰色的条形预留给文本标题和段落。
> 
> 
> 随着实际内容的逐渐加载到浏览器中，这些灰色占位符将被实际的图片和文本内容替换。
> 
> 
> ### 结论
> 
> 
> 骨架屏是一种非常有效的用户界面技术，尤其适用于网络速度较慢或数据处理较慢的应用场景，能显著提升用户的等待体验和整体满意度。通过合理设计和实现，开发者可以利用骨架屏减少用户流失，提升应用的专业感和友好感。


## 我们要实现的效果


![](https://img2023.cnblogs.com/blog/2339932/202409/2339932-20240930171540983-1082732291.gif)


上面是一个 内容展示的卡片，下面的是 加载中状态的该卡片的骨架图


## 如何实现呢？


### 1\. 先定义卡片部分 ui 代码


一个 `Column` 中有三行元素，分别是 **第一行: 图片** 、**第二行: 卡片标题** 、**第三行: 头像 昵称 浏览量**



```
class StarCard extends StatelessWidget {
  const StarCard({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 180,
      clipBehavior: Clip.hardEdge,
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(4),
      ),
      child: Column(
        children: [
          SizedBox(
            width: 180,
            child: AspectRatio(
              aspectRatio: 9 / 11,
              child: Image.network('https://pic1.zhimg.com/80/v2-fc35089cfe6c50f97324c98f963930c9_720w.jpg', fit: BoxFit.cover),
            ),
          ),
          Padding(
            padding: EdgeInsets.all(8),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  '魔法少女李知恩！！！',
                  style: TextStyle(fontSize: 15, color: Colors.black, fontWeight: FontWeight.w500),
                  maxLines: 2,
                  overflow: TextOverflow.ellipsis,
                ),
                SizedBox(height: 8),
                Row(
                  children: [
                    ClipRRect(
                      borderRadius: BorderRadius.circular(10),
                      child: Image.network(
                        'https://pic1.zhimg.com/80/v2-1956eeb2c894f1785362411aa306f882_1440w.webp?source=1def8aca',
                        height: 20,
                        width: 20,
                        fit: BoxFit.cover,
                      ),
                    ),
                    SizedBox(width: 4),
                    Text('是 IU 吖', style: TextStyle(fontSize: 12, color: const Color(0xff86909C))),
                    const Spacer(),
                    Text('21 浏览', style: TextStyle(fontSize: 12, color: const Color(0xff86909C))),
                  ],
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}


```

### 2\. 引入 `shimmer` 包


在 `pubspec.yaml` 中加入



```
dependencies:
  shimmer: ^3.0.0

```

* 该 package 的文档在这: [https://pub.dev/packages/shimmer](https://github.com)


其作用是为我们的元素加上 闪动 流光 效果，类似于一道光照射到一把光滑的宝剑上，随着宝剑角度发生变化 光的反射发生位移的现象


我们使用它的 `fromColors` 构造器，先随便放进去一个 `Container` 试试效果:



```
Shimmer.fromColors(
  baseColor: Colors.orange,
  highlightColor: Colors.blue,
  child: Container(
	color: Colors.white,
	height: 180,
	width: 180,
  ),
)

```

![](https://img2023.cnblogs.com/blog/2339932/202409/2339932-20240930171540505-725097769.png)


效果还不错，就是配色有点丑


调整下颜色，在用来包裹下我们刚刚定义的 `StarCard` :



```
Shimmer.fromColors(
  baseColor: Colors.grey[300]!, // 骨架基色
  highlightColor: Colors.grey[100]!, // 骨架高亮色
  child: StarCard(),
),

```

![](https://img2023.cnblogs.com/blog/2339932/202409/2339932-20240930171540080-116750524.png)


诶？怎么跟我们要实现的效果有点出入？


这是因为 `StarCard` 的根组件是一个带有颜色的 `Container`



```
return Container(
  width: 180,
  clipBehavior: Clip.hardEdge,
  decoration: BoxDecoration(
	color: Colors.white,
	borderRadius: BorderRadius.circular(4),
  ),
  ...
);

```

而这样 **Shimmer** 效果便会被加到整个跟组件上，`child` 也就看不到 **Shimmer** 了。那我们将根组件的 `color` 属性移除试试呢：


![](https://img2023.cnblogs.com/blog/2339932/202409/2339932-20240930171539693-638002316.png)


嗯...... 底层的元素确实展示出来了，不过我们所期望的并不是展示出文字和数据啊，况且这个时候我们还尚未拿到服务器返回给我的的数据


### 3\. Magic symbol


这个时候我们就需要用到一个 **魔法符号** ，不过在引入之前，先分离下组件：


`StarCard` 需要一个构造函数，里面接收一个从服务端 反序列化 来的 **model** ；再新定义一个 `StarCardSkeleton` 组件，他有一个无参构造器，用作 `StarCard` 的骨架图；也就是说在获取到数据之前，我们使用一个 `StarCardSkeleton` 来占 `StarCard` 的位，获取到数据之后使用 `StarCard` 来展示真实的数据，代码如下：



```
class StarCard extends StatelessWidget {
  const StarCard({super.key, required this.starModel});

  final StarModel starModel;

  @override
  Widget build(BuildContext context) {...}
}

class StarCardSkeleton extends StatelessWidget {
  const StarCardSkeleton({super.key});

  @override
  Widget build(BuildContext context) {...}
}

```

现在该回归正题了，我们要引入的 **魔法符号** 就是：



```
▆

```

这是一个 **全宽 纯色** 占位符，数个 `▆` 连起来，配合加粗 `fontWeight` 可以实现我们想要的 **一道长条色块** 的效果，且要比定义 `SizedBox` 少改动更多代码，用它来代替**Shimmer** 文本再合适不过了


我们先将 `StarCard` build 中的代码全部拷贝到 `StarCardSkeleton` 里面，并改动 **卡片标题** 、**用户昵称** 、**浏览量** `Text` 中的文字为自定义数量的 `▆`



```
class StarCardSkeleton extends StatelessWidget {
  const StarCardSkeleton({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 180,
      clipBehavior: Clip.hardEdge,
      decoration: BoxDecoration(
        // color: Colors.white,
        borderRadius: BorderRadius.circular(4),
      ),
      child: Column(
        children: [
          SizedBox(
            width: 180,
            child: AspectRatio(
              aspectRatio: 9 / 11,
              child: Container(color: Colors.white),
            ),
          ),
          Padding(
            padding: EdgeInsets.all(8),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  '▆▆▆▆▆▆',
                  style: TextStyle(fontSize: 15, fontWeight: FontWeight.w900),
                  maxLines: 2,
                  overflow: TextOverflow.ellipsis,
                ),
                SizedBox(height: 8),
                Row(
                  children: [
                    Container(
                      width: 20,
                      height: 20,
                      decoration: BoxDecoration(shape: BoxShape.circle, color: Colors.white),
                    ),
                    SizedBox(width: 4),
                    // NickNameText(articleData.nickName, views: articleData.playTimes),
                    Text('▆▆▆', style: TextStyle(fontSize: 13, fontWeight: FontWeight.w900)),
                    const Spacer(),
                    Text('▆▆', style: TextStyle(fontSize: 12, fontWeight: FontWeight.w900)),
                  ],
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

```

如果想进一步优化渲染性能，可以把 `Image` 换成带背景色的 `Container`


再看下效果呢


![](https://img2023.cnblogs.com/blog/2339932/202409/2339932-20240930171539361-514072484.png)


完美！简直一模一样！


## 抽离出 **Simmer** 组件


为了方便组件复用，可以将 **Shimmer** 封装出来



```
/// 骨架屏闪烁
class BaseShimmer extends StatelessWidget {
  const BaseShimmer({super.key, required this.child});

  final Widget child;

  @override
  Widget build(BuildContext context) {
    return Shimmer.fromColors(
      baseColor: Colors.grey[300]!, // 骨架基色
      highlightColor: Colors.grey[100]!, // 骨架高亮色
      child: child,
    );
  }
}

```

再次用到可以直接：`BaseShimmer(child: StarCardSkeleton())`


## 拓展


这章的标题是 **骨架屏** ，为什么从头到尾一直再讲怎么生成一个骨架图呢？


先别急骂标题党！


所谓的 **骨架屏** 不就是一张一张的骨架图拼出一个屏幕，不就是一个骨架屏 嘛


![](https://img2023.cnblogs.com/blog/2339932/202409/2339932-20240930171538906-329670550.png)



```
BaseShimmer(
	child: MasonryGridView.count(
		padding: EdgeInsets.only(bottom: 10, left: 12, right: 12, top: 8),
		physics: const NeverScrollableScrollPhysics(),
		itemCount: 9,
		mainAxisSpacing: 5,
		crossAxisSpacing: 5,
		crossAxisCount: 2,
		itemBuilder: (BuildContext context, int index) {
		  return StarCardSkeleton();
		}),
	)

```

这里用了 `flutter_staggered_grid_view` 包，这个包还可以做出卡片高度不一的瀑布流布局效果


![](https://img2023.cnblogs.com/blog/2339932/202409/2339932-20240930171537876-950909172.png)



> 注：使用 **BaseShimmer** 包裹整个 `GridView` 或 `ListView` 比包裹单个的 `Card` 效果要更好哟\~


## 风险


经过测试发现 在不同的设备上、或者使用了自定义字体，`▆▆▆` 之间会出现微小间距，无论将 `fontWeight` 设置为多大都无法避免，这时只能将 `▆` 方案换为带颜色的 `Container` 来解决。不过 `SkeletonScreen` 在页面停留的时间通常不会太长，这点就要看团队内部的取舍了


 本博客参考[westworld加速](https://tianchuang88.com)。转载请注明出处！

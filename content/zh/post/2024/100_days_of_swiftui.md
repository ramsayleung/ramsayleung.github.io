+++
title = "100 Days of SwiftUI"
date = 2024-04-09T19:04:00-07:00
lastmod = 2025-01-09T20:04:57-08:00
tags = ["swift", "swiftui", "ios"]
categories = ["IOS开发"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 缘起 {#缘起}

我花了半年多的时间，在闲暇时间，学习了苹果的Swift语言和SwiftUI框架，想体验下IOS开发，再看下有没有机会通过写软件来做点副业。 <br/>

先花了大概3个月时间，通过阅读 [The Swift Programming Language](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/guidedtour/) 这本官方电子书[^fn:1]来学习Swift这门语言，又花了接近4个月的时候来学习 [100 Days of SwiftUI](https://www.hackingwithswift.com/100/swiftui) 这门课程[^fn:2]，每天花费1到2小时来学习一课，总共100课，所以顾名思义叫 100 Days of SwiftUI, 课程非常新且好，讲师功力深厚，课讲得深入浅出，娓娓道来。 <br/>

每完成一课，就在Twitter上发一条推文，今天刚好把第100天的推文发了. <br/>

{{< figure src="/ox-hugo/tweet_day_100.png" >}} <br/>

{{< figure src="/ox-hugo/tweet_day_99.png" >}} <br/>

{{< figure src="/ox-hugo/tweet_day_96.png" >}} <br/>

今天是结课之日，我通过了结课的考试，总分100分，考了91分，喜提课程证书一枚. <br/>

{{< figure src="/ox-hugo/100_days_of_swiftui_certificate.jpg" >}} <br/>

在整个课程中，我写了19个IOS App(虽说大部分是功能简单的App), 源码也基本放在[ GitHub](https://github.com/ramsayleung?tab=repositories&q=&type=&language=swift&sort=)&nbsp;[^fn:3]上了，不过所有的App都没有上架App Store，因为我还没有给苹果交税(99美刀的开发者注册费). <br/>

经过这100节课和19个APP的训练，我自觉已经掌握了使用Swift和SwiftUI的基础开发技能，算是个入门的IOS开发了, 现在我可以说自己是前端，后端，数据开发，IOS开发都搞过的全栈(~~干~~)工程师了（不是） <br/>

但是在苹果对SwiftUI开发思路做出改变之前，我SwiftUI之旅可能就先到此为止了，原因下文再谈 <br/>


## <span class="section-num">2</span> Swift 初体验 {#swift-初体验}

Swift 是由LLVM之父 [Chris Lattner](https://en.wikipedia.org/wiki/Chris_Lattner)&nbsp;[^fn:4]在2010开始开发，在2014年的WWDC苹果开发者大会正式推出的一门编程语言。 <br/>

按照官方的说法，Swift从 Objective-C, Rust, Haskell, Ruby, Python, C#身上都有不同程度的借鉴和学习。 <br/>

因为我对上面提到的语言多少有涉猎，所以学习Swift起来基本没有什么困难, `Optional`, `Error Handling`, `Result`, `Generic`, `Enumerations`, `Protocol` 这些概念都和Rust的大同小异。 <br/>

又是由LLVM之父来操刀，所以语言本身也设计得很优雅. <br/>

让我眼前一亮的可能是借鉴自 [C# Extension Methods](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)&nbsp;[^fn:5]的 `extension` 功能 , 可以对已有的 class, enum 或者是 protocol 类型增加新的函数，也就是在不修改源码的情况下，扩展已有的功能. <br/>

例如，以下的代码就可以扩展内置的 `Double` 类型, 实现以米为单位，进行千米, 厘米，毫米，公尺的转换: <br/>

```swift
extension Double {
    var km: Double { return self * 1_000.0 }
    var m: Double { return self }
    var cm: Double { return self / 100.0 }
    var mm: Double { return self / 1_000.0 }
    var ft: Double { return self / 3.28084 }
}
let oneInch = 25.4.mm
print("One inch is \(oneInch) meters")
// Prints "One inch is 0.0254 meters"
let threeFeet = 3.ft
print("Three feet is \(threeFeet) meters")
// Prints "Three feet is 0.914399970739201 meters"
```

总体而言, Swift是一门吸收了众多PL理论的现代编程语言, 官方说支持Linux，Windows，MacOS等多个平台，不过我估计大多是在MacOS上用来写IOS和Mac应用 <br/>


## <span class="section-num">3</span> SwiftUI {#swiftui}

SwiftUI 使用的声明式语法，让开发者写页面布局和效果变得简洁清晰, 例如通过 `VStack`, `HStack`, `ZStack` 就可以实现X轴，Y轴，和Z轴方向的布局 <br/>

例如下面这个就是通过 `ZStack` 几行代码实现的叠加效果: <br/>

```swift
    let colors: [Color] =
    [.red, .orange, .yellow, .green, .blue, .purple]


var body: some View {
    ZStack {
        ForEach(0..<colors.count) {
            Rectangle()
                .fill(colors[$0])
                .frame(width: 100, height: 100)
                .offset(x: CGFloat($0) * 10.0,
                        y: CGFloat($0) * 10.0)
        }
    }
}
```

{{< figure src="/ox-hugo/zstack_rectangle.png" >}} <br/>

除了声明式语法之外，SwiftUI让人赏心悦目的就是动画。好的动画在App里面绝对能起到画龙点睛的作用，而SwiftUI的内置动画已经非常强大了，下面就是使用内置动画实现的动画效果: <br/>

```swift
struct ContentView: View {
    @State private var dragAmount = CGSize.zero
    @State private var enable = false
    let letters = "Hello, World"
    var body: some View{
        HStack(spacing: 0) {
            ForEach(0..<letters.count, id: \.self) { index in
                Text(String(letters[letters.index(letters.startIndex, offsetBy: index)]))
                    .padding(5)
                    .font(.title)
                    .background(enable ? .green : .blue)
                    .offset(dragAmount)
                    .animation(.linear.delay(Double(index) / 20), value: dragAmount)
            }
        }.gesture(
            DragGesture()
                .onChanged {
                    dragAmount = $0.translation
                }
                .onEnded { _ in
                    dragAmount = CGSize.zero
                    enable.toggle()
                }
        )
    }
}
```

{{< figure src="/ox-hugo/drag_animation.gif" >}} <br/>

```swift
  struct HeartBeatView: View {
    @State private var animationAmount = 1.0
    var body: some View {
        Button("SOS"){
        }
        .padding(50)
        .background(.red)
        .foregroundColor(.white)
        .clipShape(Circle())
        .overlay(
            Circle()
                .stroke(.red)
                .scaleEffect(animationAmount)
                .opacity(2 - animationAmount)
                .animation(.easeOut(duration: 1)
                    .repeatForever(autoreverses: false), value: animationAmount)
        )
        .onAppear {
            animationAmount = 2
        }
    }
}
```

{{< figure src="/ox-hugo/hearbeat.gif" >}} <br/>

而Xcode 15新增的预览功能也很好用，可以让开发者不需要启动iPhone模拟器就能预览页面效果，节省了非常多的等待时间。 <br/>

{{< figure src="/ox-hugo/xcode_preview.png" >}} <br/>


## <span class="section-num">4</span> 问题 {#问题}

听起来好像很美好: IDE新功能好用，编程语言优雅, UI框架简洁好用; 但是苹果的开发思路却有问题： 苹果开发的SwiftUI不向后兼容老版本的IOS。 <br/>

SwiftUI大部分功能都是只支持IOS16及以后的版本，而苹果新出来的数据持久框架 `SwiftData` 甚至只支持IOS17, <br/>
更离谱的是，SwiftUI的 BugFix 也只支持高版本IOS, 这就意味着用户不升级IOS版本，甚至SwiftUI的bug开发者都没法修复。 <br/>

我自己的手机也只更新到IOS16，所以我时常会遇到我自己写的App没法运行到我自己手机上的情况。 <br/>

不支持旧版本的IOS就让一大批的开发者和公司都没有动力去使用SwiftUI: <br/>

对于开发新应用的开发者而言，只支持IOS17就意味着会流失一大群使用IOS16及以下版本的用户， <br/>
而对于拥有存量用户的公司而言，更没有动力去使用SwiftUI，用了之后，旧版本IOS的用户可能直接无法打开应用。 <br/>

因此SwiftUI就陷入了一个尴尬的境地，东西做得好，但是不会有人用; <br/>

没有人自然就不用有人分享，宣传这门技术，自然就导致相关的学习资料非常匮乏, 进一步加深了初学者的学习难度; <br/>

开发遇到问题连懂的人都不用，官方文档写了又约等于没有写, 直接劝退初学者，恶性循环。 <br/>

又因为接受SwiftUI的开发者还不多，苹果版本迭代起来更加肆无忌惮，新版本又引入一堆的Breaking change，导致开发者更新版本非常痛苦. <br/>

另外一个问题就是SwiftUI与苹果现有框架整合得不够好，如 `CoreImage` 框架，顾名思义是用来作图片处理. <br/>

但之前是使用Objective-C写的，通过SwiftUI来调用，就会变成相当恶心，需要把Swift的数据结构传换成Objective-C来处理, 如： <br/>

```swift
func applyProcess(){
    guard let outputImage = currentFilter.outputImage else {return}
    guard let cgImage = context.createCGImage(outputImage, from: outputImage.extent) else{return}
    let uiImage = UIImage(cgImage: cgImage)
    processedImage = Image(uiImage: uiImage)
}

func loadImage() {
    Task{
        guard let imageData = try await selectedItem?.loadTransferable(type: Data.self) else {return}

        guard let inputImage = UIImage(data: imageData) else {return}

        let beginImage = CIImage(image: inputImage)
        currentFilter.setValue(beginImage, forKey: kCIInputImageKey)
        applyProcess()
    }
}
```

把 `CoreImage` 框架的 `CIImage` 转成 `CoreGraphics` 框架的 `CGImage`, 然后再把 `CGImage` 转换成 `UIKit` 框架 `UIImage`, 然后再转换回SwiftUI 内置的 `Image` 类型, 可谓是相当麻烦了. <br/>

但是对比SwiftUI只支持高版本的问题，Objective-C和Swift的互操作问题也只能算是恶心，但是起码有解决方法，对于前者，开发者是完全没法自行解决. <br/>


## <span class="section-num">5</span> 总结 {#总结}

过了一把野生IOS开发的瘾，但是除非是苹果愿意让SwiftUI支持低版本的IOS， <br/>
不然我是没有太大意愿继续使用SwiftUI来开发IOS了，受众比较有限了。 <br/>

想要支持低版本的IOS，就只能走UIKit和Objective-C这条历史老路，我对此着实是望而生畏，有空还是学习点其他有趣的东西。 <br/>


<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

[^fn:1]: <https://docs.swift.org/swift-book/documentation/the-swift-programming-language/guidedtour/> <br/>
[^fn:2]: <https://www.hackingwithswift.com/100/swiftui> <br/>
[^fn:3]: <https://github.com/ramsayleung?tab=repositories&q=&type=&language=swift&sort>= <br/>
[^fn:4]: <https://en.wikipedia.org/wiki/Chris_Lattner>  <br/>
[^fn:5]: <https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods>  <br/>

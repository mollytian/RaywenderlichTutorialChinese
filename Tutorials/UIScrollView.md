原文来自：[Raywenderlich](https://www.raywenderlich.com/122139/uiscrollview-tutorial)

# UIScrollView 教程：起步入门
UIScrollView 是iOS里面最多变化也是最有用的控件之一。它是非常流行的UITableView的基础，非常适合用来展现比设备屏幕尺寸更大的内容。这篇教程将通过做一个和原生照片非常类似的 app 来学习这个控件：
* 如何使用 UIScrollView 来缩放一个很大的图片
* 在所放的同时，保证 UIScrollView 的内容是居中的
* 怎样把 UIScrollView 的垂直滑动和 Auto Layout 一起用
* 如何保证在显示键盘的时候，文本输入框始终可见
* 如何使用 UIPageViewController 和 UIPageControl 来实现翻页功能
这篇教程假定你已经对Swift和iOS编程有点熟悉了。如果你是一个完全的新手，你最好先学习一下其他的基础教程。

#开始吧
点击 [这里](https://koenig-media.raywenderlich.com/uploads/2016/01/PhotoScroll_Starter.zip) 下载起始项目。在Xcode中打开，并运行：
![](http://upload-images.jianshu.io/upload_images/4727133-b12625b075f0fdcf.gif?imageMogr2/auto-orient/strip)

当选中一张照片之后，照片被切割了，因为你的手机不够大，你看不到完整的图片。你所想要的是图片能够适配手机屏幕的大小，然后可以放大缩小，就像在照片APP里一样。
你能修复它吗？可以的！

#滑动、缩放一张图片
这篇教程的第一件事就是要教你怎么使用 UIScrollView 来怎样建立一个 scroll view，好让用户能够缩放并滑动。
首先，你要添加一个 Scroll View. 打开 Main.storyboard, 从 Object Library 里拖一个 Scroll View 并放在 Document Outline里，Zoomed Photo View Controller Scene 之下。 把 Image View 移到你刚加的 Scroll View 下面。你的 Document  Outline应如下图所示：
![](https://koenig-media.raywenderlich.com/uploads/2016/01/Screen-Shot-2016-01-05-at-8.42.59-PM.png)

看见那个红点了嘛？ Xcode 这时候在提醒你，你的 Auto Layout 的规则没有定义好。我们来修复它，首先选中 Scroll View。点击 storyboard 底部的 Pin 按钮，添加四个新的约束：top，bottom，leading，trailing spaces。不要勾选 Constrain to margins 选项，把所有约束的值设为0。
![](https://koenig-media.raywenderlich.com/uploads/2016/01/Screen-Shot-2016-01-06-at-8.10.23-PM-559x500.png)

现在选中 Image View， 然后添加相同的四个约束。
选中 Zoomed Photo View Controller，然后选择 Editor\Resolve Auto Layout Issues\Update Frames，这样就没有 Auto Layout 的警告了。
跑起来：
![](https://koenig-media.raywenderlich.com/uploads/2015/12/starter2.gif)

正是有了 Scroll View，你现在可以看到完整大小的图片了。但是如果你想把图片缩放到手机屏幕大小该怎么办呢？或者是你想缩放图片?
准备好来整点儿代码了吗？
打开 ZoomedPhotoViewController.swift，在类的声明里，添加下面这些 outlet。
~~~swift
@IBOutlet weak var scrollView: UIScrollView!
@IBOutlet weak var imageViewBottomConstraint: NSLayoutConstraint!
@IBOutlet weak var imageViewLeadingConstraint: NSLayoutConstraint!
@IBOutlet weak var imageViewTopConstraint: NSLayoutConstraint!
@IBOutlet weak var imageViewTrailingConstraint: NSLayoutConstraint!
~~~

回到 Main.storyboard里，把 Scroll View 和 Zoomed View Controller 连接起来，通过把 它和scrollView outlet 连起来，而且把 Zoomed View Controller 设为 Scroll View 的 delegate（代理）。同时，像如下图所示把 Zoomed View Controller 的新的约束和 Document Outline 里的约束连起来。
![](https://koenig-media.raywenderlich.com/uploads/2016/01/Screen-Shot-2016-01-05-at-8.53.38-PM.png)
现在，你可以来写点儿代码了。在 ZoomedPhotoViewController.swift 里面，用 extension 的形式添加下面的 UIScrollView 的实现：
~~~swift
extension ZoomedPhotoViewController: UIScrollViewDelegate {
  func viewForZoomingInScrollView(scrollView: UIScrollView) -> UIView? {
    return imageView
  }
}
~~~
这是整个 scroll view 能够滑动的核心。你在告诉它，当 scroll view 被缩放的时候，哪一个 view （视图）应该变大或变小。在这里，你告诉它，应该是 imageView
现在，把下面  updateMinZoomScaleForSize(_:) 的实现加到 ZoomedPhotoViewController 类里：
~~~swift
private func updateMinZoomScaleForSize(size: CGSize) {
  let widthScale = size.width / imageView.bounds.width
  let heightScale = size.height / imageView.bounds.height
  let minScale = min(widthScale, heightScale)  

  scrollView.minimumZoomScale = minScale

  scrollView.zoomScale = minScale
}
~~~
你需要计算出 scroll view 的最小的缩小程度。缩放程度为一的意思是这个内容将会以原本的大小呈现。小于一，内容会被缩小，大于一内容就会被放大。通过计算要缩小多少才能使得图片完全能在 scroll view 的边界里面来得到缩小程度。这个程度可以让你在缩到最小看到整张图片。注意，最大缩放程度默认是1。如果你把这个调的太高的话图片会看起来不清晰。
把初始缩放程度设为最小缩放程度，这样图片一开始就是完全缩小了。
最后，每次 controller 更新它的 subview （子视图）的时候，我们都要更新最小缩放程度。
~~~swift
override func viewDidLayoutSubviews() {
  super.viewDidLayoutSubviews()

  updateMinZoomScaleForSize(view.bounds.size)
}
~~~
跑一下程序，你会得到下面的结果：
![](https://koenig-media.raywenderlich.com/uploads/2015/12/starter3.gif)
你可以在竖直模式下放大缩小这个图片来填充整个屏幕。但是有几点晓得问题：
* 这个图片总是在屏幕的最上方。它应该在中间
* 如果你把手机横过来，你的视图没有重新定义大小
还在 ZoomedPhotoViewController.swfit 里面实现 updateConstraintsForSize(size:)  函数来修复这些问题：
~~~swift
private func updateConstraintsForSize(size: CGSize) {   

  let yOffset = max(0, (size.height - imageView.frame.height) / 2)
  imageViewTopConstraint.constant = yOffset
  imageViewBottomConstraint.constant = yOffset

  let xOffset = max(0, (size.width - imageView.frame.width) / 2)
  imageViewLeadingConstraint.constant = xOffset
  imageViewTrailingConstraint.constant = xOffset

  view.layoutIfNeeded()
}
~~~

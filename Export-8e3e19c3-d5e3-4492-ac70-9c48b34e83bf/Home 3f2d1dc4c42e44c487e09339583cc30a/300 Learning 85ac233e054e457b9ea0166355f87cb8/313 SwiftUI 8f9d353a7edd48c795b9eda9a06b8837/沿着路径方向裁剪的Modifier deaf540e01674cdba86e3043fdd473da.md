# 沿着路径方向裁剪的Modifier

- SwiftUI lets us draw only part of a stroke or fill for a shape using its **trim()** modifier, which takes two parameters: a start value and an end value, both stored as a **CGFloat** between 0 and 1.

For example, if you wanted a semicircle you would write this:

SwiftUI draws its shapes so that 0 degrees is directly to the right, so if you want to change that so 0 degrees is directly up you should apply a **rotationEffect()** modifier.

可以用这个命令沿着Stroke的方向进行裁剪

![Circle().png](%E6%B2%BF%E7%9D%80%E8%B7%AF%E5%BE%84%E6%96%B9%E5%90%91%E8%A3%81%E5%89%AA%E7%9A%84Modifier%20deaf540e01674cdba86e3043fdd473da/Circle().png)
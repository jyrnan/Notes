# 基于ForEach实现滑动删除onDelete

> 主要原因是我将介绍的 `onDelete`处理器只能使用 `ForEach`
> 

```swift
List {
    ForEach(restaurants) { restaurant in
        BasicImageRow(restaurant: restaurant)
    }
    .onDelete { (indexSet) in
        self.restaurants.remove(atOffsets: indexSet)
    }
}
.listStyle(.plain)
```
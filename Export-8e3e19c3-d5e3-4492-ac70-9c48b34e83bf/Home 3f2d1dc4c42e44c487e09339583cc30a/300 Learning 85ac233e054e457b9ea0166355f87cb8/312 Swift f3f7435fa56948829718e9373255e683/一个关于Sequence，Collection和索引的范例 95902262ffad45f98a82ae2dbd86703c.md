# 一个关于Sequence，Collection和索引的范例

```swift
import UIKit
import Foundation

/// List 集合类型的私有实现细节，开始新建一个可以用于索引具体内容的Node数据形式
///这个数据其实会封装在后续的ListIndex当中，所以有一个fileprivate的修饰。
fileprivate enum ListNode<Element> {
    case end
    indirect case node(Element, next: ListNode<Element>)
    func cons(_ x: Element) -> ListNode<Element> {
        return .node(x, next: self)
    }
}

///新建一个用来可以作为索引的数据结构，
///这个封装了ListNode在里面作为具体内容，并通过Node数据内部的next属性可以实现链表的指针效果，从而将各个ListIndex实例串起来
///实际上这是一个既是索引，同时又是把数据封装在里面了
public struct ListIndex<Element>: CustomStringConvertible {
    //这是真正的数据内容
    fileprivate let node: ListNode<Element>
    //这里是在node基础上增加一个tag用来作为索引标记，是作为索引的数据（索引一般推荐为整数）
    fileprivate let tag: Int
    
    public var description: String {
        return "ListIndex\(tag)"
    }   
}

//实现Comparable协议需要的两个函数
extension ListIndex: Comparable {
    public static func < (lhs: ListIndex<Element>, rhs: ListIndex<Element>) -> Bool {
        return lhs.tag > rhs.tag //因为这个链表的索引是最末尾（也就是初始位置）为0，索引这里是用的大于号来判断
    }
    
    public static func == (lhs: ListIndex<Element>, rhs: ListIndex<Element>) -> Bool {
        return lhs.tag == rhs.tag
    }   
}

///顶层直接可见的数据结构，通过让其符合Collection协议，所以需要对其Index做定义。
///而他的Index类型符合ListIndex，一方面充当了index来索引，而每个Index里面其实就封装了真正的数据：ListNode
public struct List<Element>: Collection {
   // Index 的类型可以被推断出来，不过写出来可以让代码更清晰一些
    public typealias Index = ListIndex<Element>
    
    public let startIndex: Index
    public let endIndex: Index
    
    public subscript(position: Index) -> Element {
        switch position.node {
        case .end: fatalError("Subscript out of range")
        case let .node(x, _): return x
        }
    }
    
    public func index(after idx: Index) -> Index {
        switch idx.node {
        case .end: fatalError("Subscript out of range")
        //case let .node(_, next)
        case .node( _, let next): //这个赋值（读取enum的associateValue）表达形式有多种,例如上一行，需要熟记
            return Index(node: next, tag: idx.tag - 1)
        }
    }
}

///实现用array的形式来生成List
extension  List: ExpressibleByArrayLiteral {
    public init(arrayLiteral elements: Element...) {
        startIndex = ListIndex(node: elements.reversed().reduce(.end){
            patialList, element in
            patialList.cons(element)
        }, tag: elements.count)
        endIndex = ListIndex(node: .end, tag: 0)
    }  
}
///
extension List: CustomStringConvertible {
    public var description: String {
        let elements = self.map{
            String(describing: $0)
        }.joined(separator: ",")
        return "List:(\(elements))"
    }
}

let list: List = ["one", "two", "three"] //List:(one,two,three)
list.first//"one"
list.firstIndex(of: "two") //ListIndex3
```

[https://www.notion.so](https://www.notion.so)
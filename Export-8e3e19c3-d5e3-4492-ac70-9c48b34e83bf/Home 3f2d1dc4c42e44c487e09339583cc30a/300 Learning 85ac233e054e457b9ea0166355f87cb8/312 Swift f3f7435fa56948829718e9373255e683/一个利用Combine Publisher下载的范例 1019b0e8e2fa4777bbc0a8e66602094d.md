# 一个利用Combine/ Publisher下载的范例

```swift
import UIKit
import Combine

class ViewModel: ObservableObject {
    @Published var image: UIImage?
    
    let url = "https://www.raywenderlich.com/rails/active_storage/blobs/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBBWDg9IiwiZXhwIjpudWxsLCJwdXIiOiJibG9iX2lkIn19--7c69b66c5cdec3db3a2cbf56bdab336c26cca650/path-ios-beginner@2x.png"
    
    private var fetchImageCancellable: AnyCancellable?
    
    func fetchImage() {
            image = nil
        if let url = URL(string: url) {
            fetchImageCancellable?.cancel()
            fetchImageCancellable = URLSession.shared.dataTaskPublisher(for: url)
                .map{data, usrResponse in UIImage(data: data)}
                .receive(on: DispatchQueue.main)
                .replaceError(with: nil)
                .assign(to: \.image, on: self)
        }
    }
}
```
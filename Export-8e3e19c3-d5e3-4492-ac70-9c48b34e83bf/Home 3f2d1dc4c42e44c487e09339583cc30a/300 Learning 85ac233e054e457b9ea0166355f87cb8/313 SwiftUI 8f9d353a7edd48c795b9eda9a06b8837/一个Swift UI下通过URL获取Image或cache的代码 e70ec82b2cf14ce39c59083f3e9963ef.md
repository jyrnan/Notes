# 一个Swift UI下通过URL获取Image或cache的代码

特别值得关注的是最后的NSCache的使用

```swift
struct ImageContentView : View {
    @State var url = "https://link-to-image"
    var body: some View {
        VStack {
            ImageView(imageUrl: url)
        }
    }
}
struct ImageView: View {
    @ObservedObject var remoteImageModel: RemoteImageModel
    
    init(imageUrl: String?) {
        remoteImageModel = RemoteImageModel(imageUrl: imageUrl)
    }
    
    var body: some View {
        Image(uiImage: remoteImageModel.displayImage ?? UIImage(named: "default-image-here")!)
            .resizable()
    }
}

class RemoteImageModel: ObservableObject {
    @Published var displayImage: UIImage?
    var imageUrl: String?
    var cachedImage = CachedImage.getCachedImage()
    
    init(imageUrl: String?) {
        self.imageUrl = imageUrl
        if imageFromCache() {
            return
        }
        imageFromRemoteUrl()
    }
    
    
    func imageFromCache() -> Bool {
        guard let url = imageUrl, let cacheImage = cachedImage.get(key: url) else {
            return false
        }
        displayImage = cacheImage
        return true
    }
    
    func imageFromRemoteUrl() {
        guard let url = imageUrl else {
            return
        }
        
        let imageURL = URL(string: url)!
        
        URLSession.shared.dataTask(with: imageURL, completionHandler: { (data, response, error) in
            if let data = data {
                DispatchQueue.main.async {
                    guard let remoteImage = UIImage(data: data) else {
                        return
                    }
                    
                    self.cachedImage.set(key: self.imageUrl!, image: remoteImage)
                    self.displayImage = remoteImage
                }
            }
        }).resume()
    }
}

class CachedImage {
    var cache = NSCache<NSString, UIImage>()
    
    func get(key: String) -> UIImage? {
        return cache.object(forKey: NSString(string: key))
    }
    
    func set(key: String, image: UIImage) {
        cache.setObject(image, forKey: NSString(string: key))
    }
}

extension CachedImage {
    private static var cachedImage = CachedImage()
    static func getCachedImage() -> CachedImage {
        return cachedImage
    }
}
```
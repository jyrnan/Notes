# Camera and Photo tutorial by Apple

# DataModel

这是整个App的数据代码顶层，包含两个主要因素：相机和照片合集

```swift
let camera = Camera()
//打开的相册，有多种初始化方法
let photoCollection = PhotoCollection(smartAlbum: .smartAlbumUserLibrary)
```

## PhotoData

应用中的照片基本数据。这是把原始照片数据提炼后生成的适合app的基本数据结构

```swift
fileprivate struct PhotoData {
    var thumbnailImage: Image
    var thumbnailSize: (width: Int, height: Int)
    var imageData: Data
    var imageSize: (width: Int, height: Int)
}
```

## 照片处理环节

```swift
// 处理照片信息，生成PhotoData，这里可以自己定制处理结果
    private func unpackPhoto(_ photo: AVCapturePhoto) -> PhotoData? {
        guard let imageData = photo.fileDataRepresentation() else { return nil }

        guard let previewCGImage = photo.previewCGImageRepresentation(),
           let metadataOrientation = photo.metadata[String(kCGImagePropertyOrientation)] as? UInt32,
              let cgImageOrientation = CGImagePropertyOrientation(rawValue: metadataOrientation) else { return nil }
        let imageOrientation = Image.Orientation(cgImageOrientation)
        let thumbnailImage = Image(decorative: previewCGImage, scale: 1, orientation: imageOrientation)
        
        let photoDimensions = photo.resolvedSettings.photoDimensions
        let imageSize = (width: Int(photoDimensions.width), height: Int(photoDimensions.height))
        let previewDimensions = photo.resolvedSettings.previewDimensions
        let thumbnailSize = (width: Int(previewDimensions.width), height: Int(previewDimensions.height))
        
        return PhotoData(thumbnailImage: thumbnailImage, thumbnailSize: thumbnailSize, imageData: imageData, imageSize: imageSize)
    }
```

# PhotoCollection

这个是关于相册和照片操作的核心模块

基于`AssetCollection` 的`ObservableObject` 给PhotoCollectionView来提供数据。

## 三个不同的初始化方法

- **init**(albumNamed albumName: String, createIfNotFound: Bool = **false**)
    - albumName就是图库中相册的名字
    - createIfNotFound 设置成true会自动创建该名字的相册
- **init**?(albumWithIdentifier identifier: String)： identifier暂时没明白
- **init**(smartAlbum smartAlbumType: PHAssetCollectionSubtype)

### load方法

这个方法处理初始化方法中的1、3项。

每次处理后都要调用[refreshPhotoAssets()](Camera%20and%20Photo%20tutorial%20by%20Apple%205dc9bf9a5015479e90fd350120e8f354.md)

## refreshPhotoAssets()

用来进行相册重新载入后来更新PhotoCollection里面的photoAssets，这也是整个DataModel照片数据的核心

## PhotoAssetCollection

底层基于`PHFetchResult<PHAsset>`的序列，但是能提供PhotoAsset

```swift
private(set) var fetchResult: PHFetchResult<PHAsset>
```

### cache和PhotoAsset

通过一个cache来实现提供PhotoAsset。这也是一个cache的实现方式

```swift
private var cache = [Int : PhotoAsset]()
...
subscript(position: Int) -> PhotoAsset {
        if let asset = cache[position] {
            return asset
        }
        let asset = PhotoAsset(phAsset: fetchResult.object(at: position), index: position)
        cache[position] = asset
        return asset
    }
```

### 通过`PHFetchResult<PHAsset>` 来产生PHAsset的序列

`fetchResult.enumerateObjects`这是自带的方法

```swift
var phAssets: [PHAsset] {
        var assets = [PHAsset]()
        fetchResult.enumerateObjects { (object, count, stop) in
            assets.append(object)
        }
        return assets
    }
```

## PhotoAsset

### PHAsset的包装

```swift
// 这个很重要！！！通过identifier来获得Photos里面的照片？
    init(identifier: String) {
        self.identifier = identifier
        let fetchedAssets = PHAsset.fetchAssets(withLocalIdentifiers: [identifier], options: nil)
        self.phAsset = fetchedAssets.firstObject
    }
```

/

# SwiftUI框架下的iOS相册访问的示例代码

## PHPickerViewController(configuration: config)

```swift
import SwiftUI
import PhotosUI

struct PhotoPicker: UIViewControllerRepresentable {
    @Binding var selectedImage: UIImage?
    @Environment(\.presentationMode) var presentationMode

    func makeUIViewController(context: Context) -> PHPickerViewController {
        var config = PHPickerConfiguration()
        config.filter = .images
        config.selectionLimit = 1
        let picker = PHPickerViewController(configuration: config)
        picker.delegate = context.coordinator
        return picker
    }

    func updateUIViewController(_ uiViewController: PHPickerViewController, context: Context) {}

    func makeCoordinator() -> Coordinator {
        Coordinator(parent: self)
    }

    class Coordinator: NSObject, PHPickerViewControllerDelegate {
        let parent: PhotoPicker

        init(parent: PhotoPicker) {
            self.parent = parent
        }

        func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
            parent.presentationMode.wrappedValue.dismiss()

            guard let selectedImage = results.first?.itemProvider.loadObject(ofClass: UIImage.self) as? UIImage else {
                return
            }

            parent.selectedImage = selectedImage
        }
    }
}

```

可以将这个代码嵌入到你的SwiftUI视图中，然后使用`@State`变量来保存选中的图像，例如：

```swift
struct ContentView: View {
    @State var selectedImage: UIImage?

    var body: some View {
        VStack {
            if let image = selectedImage {
                Image(uiImage: image)
                    .resizable()
                    .scaledToFit()
            } else {
                Text("请选择一张照片")
            }
            Button("选择照片") {
                // 显示相册选择器
                self.showPhotoPicker = true
            }
        }
        .sheet(isPresented: $showPhotoPicker) {
            // 将PhotoPicker视图作为sheet展示
            PhotoPicker(selectedImage: $selectedImage)
        }
    }
}

```

这就是一个基本的iOS相册访问的实现，希望对你有所帮助。
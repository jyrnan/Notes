# MarkDown Picture Style Tips

图片可以放在文件相对的位置，并用 ./path来访问

以下方法试用于Jekyll中markdown生产html的网页

直接在图片后面加上对应的CSS样式即可

```markdown
![test image size](url)
```

改写为：

```markdown
![test image size](url){:class="img-responsive"}
![test image size](url){:height="50%" width="50%"}
![test image size](url){:height="100px" width="400px"}
```
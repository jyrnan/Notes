# Vapor如何response传回JSON数据

1. 创建数据类型，符合`Content`协议。content协议其实符合Codable，可以理解成就是Codable
2. 直接在Route中返回符合`Content`协议的数据。Vapor会根据返回的内容，因为符合`Content`协议，所以直接生成JSON
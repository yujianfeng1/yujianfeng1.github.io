### 第一步：安装Go

1. 访问Go语言的官方网站：https://golang.org/
2. 点击下载页面，选择适合你的操作系统的安装包（Windows、macOS或Linux）。
3. 下载并安装Go。
4. 安装完成后，打开命令行（在Windows上是命令提示符或PowerShell，在macOS或Linux上是终端），输入以下命令来检查Go是否安装成功：
   ```bash
   go version
   ```
   如果看到Go的版本信息，说明安装成功。

### 第二步：编写你的第一个Go程序

1. 打开你喜欢的文本编辑器或IDE（如Visual Studio Code、GoLand等）。
2. 创建一个新的文件，命名为`hello.go`。
3. 在文件中输入以下代码：
   ```go
   package main

   import "fmt"

   func main() {
       fmt.Println("Hello, World!")
   }
   ```
   这段代码定义了一个简单的Go程序，它输出"Hello, World!"。

### 第三步：运行你的Go程序

1. 打开命令行，切换到包含`hello.go`文件的目录。
2. 运行以下命令来编译并运行你的程序：
   ```bash
   go run hello.go
   ```
3. 如果一切正常，你应该会在命令行看到输出"Hello, World!"。

恭喜你！你已经成功编写并运行了你的第一个Go程序。在接下来的课程中，我们将深入学习Go语言的更多功能和概念。希望你继续保持学习的热情！
<!-- ##{"timestamp":1672552987}## -->
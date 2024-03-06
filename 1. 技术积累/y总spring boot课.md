#  y总spring boot课

- 基本类型
  ```java
  long d = 123444567778L;
  float e = 1.2F; // 默认为double
  double f = 1.2, g = 1.2D;
  final char s = 'A'; // 相当于C++中 const
  ```

 

- 隐式类型转化

  ```java
  int x = (int)c; // 显示
  int y = 12;
  double z = y; // 隐式
  double m = 4 * 3.3 // 默认隐式将4转为double
  int n = (int)(4 * 3.3);
  ```



- 输入输出

  ```java
  import java.util.Scanner;
  
  Scanner sc = new Scanner(System.in);
  sc.next(); // 读字符串，以空格断开
  sc.nextLine(); // 读一行
  
  import java.io.BufferedReader;
  import java.io.InputStreamReader;
  // Scanner 读入数据比较多，速度会慢
  BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
  String str = br.readLine(); // 读一行
  int x = br.read(); // 读一个字符
  
  import java.io.BufferedWriter;
  import java.io.OutputStreamWriter;
  BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
  bw.write("Hello World\n");
  bw.flush();
  ```



- 读入整数

  ```java
  int x = Integer.parseInt(br.readLine());
  
  // 一行多个整数
  String[] strs = br.readLine().split(" ");
  int x = Integer.parseInt(strs[0]);
  ```

## 如何将项目通过git进行管理

### 创建git仓库并与本地绑定

1. 本地创建密钥，打开git bash

   ```bash
   cd ~
   ssh-keygen
   ```

   此时在`.ssh`目录下生成一对公私钥

   ```bash
   cat id_rsa.pub
   // 也可能是 id_ed25519.pub 取决于所用的加密算法不同
   ```

   到github中 `"Setting" => "New SSH Key" `，将公钥复制到其中并保存。

2. 在本地系统中选择一个文件目录位置创建项目文件，右键选择`"git bash here"`

   ```bash
   git init 
   touch readme.md
   git add .
   git commit -m "创建项目"
   // 如果未配置usr信息需要执行下面两行
   git config --global usr.name "xxx"
   git config --global usr.email "xxx@xxx.com"
   ```

3. 打开github，新建一个仓库，不要勾选创建readme.md文件（否则仓库中已有内容，不会出现导向性命令提示），在第2步中的git bash中执行导向性命令如下：

   ```bash
   git remote add origin git@github.com:zipingw/kob.git
   git branch -M main
   git push -u origin main
   ```

   刷新github，此时仓库中出现readme.md，说明本地项目成功上传至github仓库。

### 如何同步Home和Company的电脑共同维护一个项目

1. 假设基于前者已经在Home的PC中完成仓库建立，此时到Company的新电脑中，类似于前者步骤1，需要在本地生成一对公私钥。

   ```bash
   ssh-keygen
   cat id_rsa.pub
   // 在github中新增SSH Key,粘贴公钥
   ```

2. 打开github仓库，找到SSH克隆地址

   ```bash
   git clone xxx
   ```

3. 对项目进行修改之后，将修改后的项目上传至远端仓库。

   ```bash
   git add .
   git commit -m "xxx"
   git push
   ```

4. 在Home的PC中，可以直接拉取（pull）远端仓库的内容以获得在Company中完成的项目最新版本。

   ```bash
   git pull
   ```


### 当项目基础是从github上克隆得到时

- 此时如果在项目上做了一定的修改，希望将其中的内容通过建立自己的仓库进行管理，需要执行以下操作：

1. 先按照建立新仓库的步骤执行一遍，会发现在 `git commit` 之后在执行下面的命令时会报错：

   ```bash
   $ git remote add origin git@github.com:zipingw/rtree.git
   # 得到错误 error: remote origin already exists.
   ```

   上述错误表明，我们在`git clone`时，本地的项目与克隆的远程仓库已经建立了关联，如下所示
   
   ```bash
   $ git remote -v
   origin  https://github.com/davidmoten/rtree.git (fetch)
   origin  https://github.com/davidmoten/rtree.git (push)
   ```

2. 所以我们需要先断开与原代码仓库的联系，再关联我们自己的代码仓库

   ```bash
   $ git remote rm origin
   $ git remote add origin git@github.com:zipingw/rtree.git
   ```

3. 此时再将本地项目内容push至远端仓库即可成功

   ```bash
   $ git push -u origin main
   ```


## Java后端与Vue前端项目框架搭建

1. IDEA新建 `Spring Initializr` ，默认源为 `start.spring.io` ，修改 `Name` 为项目名称 `backend` (`artifact` 会与此保持同名)，修改 `Group` 为 `com.kob` 其中 `kob` 为该后端项目最上层包名称。点击下一步并勾选`Web`中的`Spring Web` 和`Template Engines`中的`Thymeleaf` 后者用于实现前后端不分离
2. 


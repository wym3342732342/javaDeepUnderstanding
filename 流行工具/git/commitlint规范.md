## 一、idea + git commit template直接应用

1. 安装插件

   ![image-20211110155115638](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20211110155121.png)

2. commit时点击生成信息

   ![image-20211110155309792](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20211110155309.png)

   ![image-20211110155335695](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20211110155335.png)

3. **类型说明**

   - `type`类型：用于说明git commit的类别，只允许使用下面的标识

     - `feat`：新功能（feature）

     - `fix/to`：修复bug，可以是QA发现的BUG，也可以是研发自己发现的BUG。

       - `fix`：产生`diff`并自动修复此问题。适合于一次提交直接修复问题

       - `to`：只产生`diff`不自动修复此问题。适合于多次提交。最终修复问题提交时使用`fix`。

       - > `diff`：差分，可以理解问代码变动

     - `docs`：文档（documentation）

     - `style`：格式（不影响代码运行的变动）。

     - `refactor`：重构（即不是新增功能，也不是修改bug的代码变动）。

     - `perf`：优化相关，比如提升性能、体验。

     - `test`：增加测试。

     - `chore`：构建过程或辅助工具的变动。

     - `revert`：回滚到上一个版本。

     - `merge`：代码合并。

     - `sync`：同步主线或分支的Bug。

   - `scope`(可选)：用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

   - `subject`(必须)：是commit目的的简短描述，不超过50个字符。

     - 建议使用中文。
     - 结尾不加句号或其他标点符号。

   - `breaking changes`:突破性的变换

   - `closed issues`：关闭掉的问题

4. 效果展示

   ![image-20211110161154175](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20211110161154.png)


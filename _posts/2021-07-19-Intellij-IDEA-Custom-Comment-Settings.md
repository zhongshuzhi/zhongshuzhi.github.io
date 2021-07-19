## 设置类注释

1. 在IDEA中，进入File -> Settings -> Editor -> File and Code Templates -> Files

2. 找到Class（建议将Interface、Enum、Annotation Type一并修改），在

   `#parse（“File Header.java”）`及

   `public class ${Name}`

   间添加如下代码：

   ```java
   /**
   
   * Please enter your class description here.
   *
   
   * @version 1.0.0
   
   * @author ${USER}
   
   * @date ${DATE} ${TIME}
   */
   ```

3. 实际效果如图所示

   ![image-20210719230751775](C:\Users\ZSZ\AppData\Roaming\Typora\typora-user-images\image-20210719230751775.png)

4. 保存（Apply），此时类注释已设置完成

------

## 设置方法注释

1. 在IDEA中，进入File -> Settings -> Editor -> Live Templates

2. 点击右侧+号，添加Template Group，命名为userDefine，如图所示

   ![image-20210719230852291](C:\Users\ZSZ\AppData\Roaming\Typora\typora-user-images\image-20210719230852291.png)

3. 点击右侧+号，添加Live Templates，

   - Abbreviation 填写 `*`

   - Description 填写 `Custom comments for method in fcloud projects`

   - Template text 填写

     ```java
     *
      * Please enter the description here.
      *
      * @author $user$
      * @date $date$ $time$$param$ $return$
      */
     ```

   - option中
     
     - expand with下拉框选择**Enter**（回车键）

   整体配置如图所示

   ![image-20210719231002773](C:\Users\ZSZ\AppData\Roaming\Typora\typora-user-images\image-20210719231002773.png)

4. 此时若一切正常，edit variables应处于可点击的状态，点击，修改变量表达式（Expression）

   - user一栏Expression修改为`user()`，可从下拉框中选择

   - date一栏Expression修改为`date()`，可从下拉框中选择

   - time一栏Expression修改为`time()`，可从下拉框中选择

   - param一栏Expression修改为

     ```groovy
     groovyScript("def result = '';def params = \"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); for(i = 0; i < params.size(); i++) {if(params[i] != '')result+='* @param ' + params[i] + ((i < params.size() - 1) ? '\\r\\n ' : '')}; return result == '' ? null : '\\r\\n ' + result", methodParameters()) 
     ```

     

   - return一栏Expression修改为

     ```groovy
     groovyScript("return \"${_1}\" == 'void' ? null : '\\r\\n * @return ' + \"${_1}\"", methodReturnType()) 
     ```

   整体效果如图所示

   ![image-20210719231107279](C:\Users\ZSZ\AppData\Roaming\Typora\typora-user-images\image-20210719231107279.png)

5. 保存（Apply），此时方法体注释已自定义完成



------

## 效果预览

- 若上述设置均成功，那么创建类时，类注释会自动创建

- 方法注释需先定义好方法后，在方法上一行输入`/**`，敲击**Enter（回车）**，方法注释会自动补全。

- 整体效果如图所示

  ![image-20210719231234784](C:\Users\ZSZ\AppData\Roaming\Typora\typora-user-images\image-20210719231234784.png)



--------

## 备注

- 简要介绍一下原理，类注释使用的是IDEA自带的注释补全功能，由IDEA捕获类创建的操作并在指定位置添加代码，使用的变量为IDEA预定义变量。同理，若各位想要在其他类型文件中自动创建注释，例如xml，yaml等，也可自行定义注释规则。

- 方法注释的实现原理为自定义捕获事件，本文档中自定义的捕获事件为`*+Enter（回车）`，我们输入 `* + Enter` 就能够触发模板。这也同时说明了为什么注释模板首行是一个 `*` 了，因为当我们先输入 `/*`，然后输入 `* + Enter`，触发模板，首行正好拼成了 `/**`，符合 Javadoc 的规范。

  因此在其他地方输入该事件时，也会触发该模板，若对各位开发造成困扰，可禁用该模板并自行按照该格式编写方法注释。

- 参考文档

  https://blog.csdn.net/qq_17231297/article/details/114812080
# webpack之css工程化

**问题**：平常书写css存在类名冲突，样式覆盖，及嵌套层级深，难以阅读等问题

**解决方法**：

1. 利用loader来处理css

   - css-loader用来将css转化成字符串，以便于导出
     - 如果css中有导入其他css,会将其他css内容复制在改文件中
     - 如果有引入图片，会import该图片，import内容让其他loader处理
   - style-loader用于将css-loader转成字符串的内容，通过创建style标签，插入在文档中

   
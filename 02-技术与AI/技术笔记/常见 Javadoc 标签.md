# 常见 Javadoc 标签

| 标签              | 描述                                              | 示例                                                                |
| --------------- | ----------------------------------------------- | ----------------------------------------------------------------- |
| `@author`       | 标识一个类的作者                                        | `@author description`                                             |
| `@deprecated`   | 指名一个过期的类或成员                                     | `@deprecated description`                                         |
| `{@docRoot}`    | 指明当前文档根目录的路径                                    | `Directory Path`                                                  |
| `@exception`    | 标志一个类抛出的异常                                      | `@exception exception-name explanation`                           |
| `{@inheritDoc}` | 从直接父类继承的注释                                      | `Inherits a comment from the immediate superclass.`               |
| `{@link}`       | 插入一个到另一个主题的链接                                   | `{@link name text}`                                               |
| `{@linkplain}`  | 插入一个到另一个主题的链接，但该链接显示纯文本字体                       | `Inserts an in-line link to another topic.`                       |
| `@param`        | 说明一个方法的参数                                       | `@param parameter-name explanation`                               |
| `@return`       | 说明返回值类型                                         | `@return explanation`                                             |
| `@see`          | 指定一个到另一个主题的链接                                   | `@see anchor`                                                     |
| `@serial`       | 说明一个序列化属性                                       | `@serial description`                                             |
| `@serialData`   | 说明通过 `writeObject()` 和 `writeExternal()` 方法写的数据 | `@serialData description`                                         |
| `@serialField`  | 说明一个 `ObjectStreamField` 组件                     | `@serialField name type description`                              |
| `@since`        | 标记当引入一个特定的变化时                                   | `@since release`                                                  |
| `@throws`       | 和 `@exception` 标签一样                             | `@throws exception-name explanation`                              |
| `{@value}`      | 显示常量的值，该常量必须是 `static` 属性                       | `Displays the value of a constant, which must be a static field.` |
| `@version`      | 指定类的版本                                          | `@version info`                                                   |

---

## 其他常见 Javadoc 标签

### `@see`
用于指定与当前类或方法相关的其他类、方法或文档，通常在类或方法的结尾使用。

**示例**：
```java
/**
 * This method processes a file.
 * @see java.io.File
 */
public void processFile(File file) {
    // method implementation
}
```

## 额外说明

- **`@param`** 和 **`@throws`** 标签有时可能会混淆，记住 **`@param`** 用于描述方法参数，而 **`@throws`** 用于描述方法抛出的异常。
    
- **`@see`** 常用于为当前方法或类提供相关文档、API 文档或其他资源的链接。
    
- **`{@link}`** 用于插入内联的链接，可以在 Javadoc 文档中直接生成可点击的链接，常用于类之间、方法之间或常量之间的引用。

通过合理使用这些标签，Javadoc 不仅能帮助开发者更好地理解代码，也能提升项目文档的可维护性和可读性。
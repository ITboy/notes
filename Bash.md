# Brace Expansion

From [Brace Expansion](Brace Expansion)

大括号表达式在bash中被首先展开
1. 生成任意大括号中可能出现的字符串
2. 大括号前面不能有$，避免与变量展开冲突${xxx}
3. 如果有..表明字符间的所有都可以

# Example
```bash
# 删除SuperAgent.js和SuperAgent.min.js
rm -fr SuperAgent{,.min}.js
```

```bash
# 删除1.js, 2.js, 3.js
rm -fr {1..3}.js
```

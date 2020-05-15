# Xcode 配置快捷键



```
cp /Applications/Xcode.app/Contents/Frameworks/IDEKit.framework/Resources/IDETextKeyBindingSet.plist ~/Desktop

```

## 添加配置
```
<key>GDI Commands</key>
    <dict>
        <key>GDI Insert Line Below</key>
        <string>moveToEndOfLine:, insertNewline:</string>
        <key>GDI Insert Line Above</key>
        <string>moveUp:, moveToEndOfLine:, insertNewline:</string>
        <key>GDI Move Current Line Down</key>
        <string>selectLine:, cut:, moveDown:, moveToBeginningOfLine:, insertNewLine:, paste:, moveBackward:</string>
        <key>GDI Move Current Line Up</key>
        <string>selectLine:, cut:, moveUp:, moveToBeginningOfLine:, insertNewLine:, paste:, moveBackward:</string>
        <key>GDI Delete Current Line</key>
        <string>moveToEndOfLine:, deleteToBeginningOfLine:, deleteBackward:, moveDown:, moveToEndOfLine:</string>
        <key>GDI Duplicate Current Line</key>
        <string>selectLine:, copy:, moveToEndOfLine:, insertNewline:, paste:, deleteBackward:</string>
    </dict>
```

这个dict是一组可以设置快捷键的操作。意思显而易见

GDI Insert Line Below 在当前行下面增加一空行（不管光标是否在行尾）
GDI Insert Line Above 在当前行上面增加一空行
GDI Move Current Line Down 把当前行往下移动一行
GDI Move Current Line Up 把当前行往上移动一行
GDI Delete Current Line 删除当前行
GDI Duplicate Current Line 复制当前行到下面一行



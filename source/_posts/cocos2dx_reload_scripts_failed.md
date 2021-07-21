---
title: Cocos2dx脚本热更新后无法重新加载
---

## 问题描述
下载新包热更新完成后，打开游戏内合成界面会报`attempt to call field 'getItemLevel' (a nil value)`错误，这是一个在`DTHelper.lua`中的新增的函数。而大退重进则正常运行
## 原因
问题初步定位是热更新完成时为**文件没有重新加载**  
`lua`脚本重载方式一般为
```lua
function reload(moduleName)
    package.loaded[moduleName] = nil
    return require(moduleName)
end
```
脚本确实时走了重新加载的，这个机制很简单，一般不会出问题，应该是文件读取的问题，在`Cocos2dx`中，`lua`脚本加载会走`cocos2dx_lua_loader`函数，这个函数决定从`moduleName`映射到实际文件读取并执行。在热更新完成后，打印`DTHelper.lua`的全路径
```lua
-- test code
local fileUtils = cc.FileUtils:getInstance()
dump(fileUtils:getSearchPaths(), "fileUtils:getSearchPaths")
print("DTHelper.luac", fileUtils:fullPathForFilename('app/common/DTHelper.luac'))
```
1. 第一次清空应用缓存后运行（模拟刚热更完成时）
> [LUA-print] DTHelper.luac	assets/src/app/common/DTHelper.luac
2. 第二次大退后运行（模拟已经完成热更新，直接进入游戏）
> [LUA-print] DTHelper.luac	/data/user/0/com.xxx.xxxx/files/legend1.1.0//src/app/common/DTHelper.luac

第一次测试结果与期望不符，访问的仍然时包内文件路径，猜测**可能是读取路径有缓存**，查看`fullPathForFilename`发现了端倪：[`FileUtils`](https://github.com/cocos2d/cocos2d-x/blob/95e5d868ce5958c0dadfc485bdda52f1bc404fe0/cocos/platform/CCFileUtils.cpp)缓存了文件路径，即便热更新目录已经下载了新文件，依然用的是之前加载的路径
```c++
std::string FileUtils::fullPathForFilename(const std::string &filename) const
{
    ....
    // Already Cached ?
    auto cacheIter = _fullPathCache.find(filename);
    if(cacheIter != _fullPathCache.end())
    {
        return cacheIter->second;
    }
    ....
}
```

## 解决办法
清理缓存路径后再重载脚本
```lua
cc.FileUtils:getInstance():purgeCachedEntries()
-- reload scripts
```

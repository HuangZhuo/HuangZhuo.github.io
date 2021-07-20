---
title: Cocos2dx脚本热更新后无法重新加载
---

## 原因
[`FileUtils`](https://github.com/cocos2d/cocos2d-x/blob/95e5d868ce5958c0dadfc485bdda52f1bc404fe0/cocos/platform/CCFileUtils.cpp)缓存了文件路径，即便热更新目录已经下载了新文件，依然用的是之前加载的路径
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
```lua
cc.FileUtils:getInstance():purgeCachedEntries()
-- reload scripts
```
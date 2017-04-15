title: word pattern 实现
date: 2017-04-15
tags: ["算法","leetcode","学习"]
categories:
  算法
---
## 简介 ##
[leetcode第290题](https://leetcode.com/problems/word-pattern/#/description)

## 代码 ##
### go ###
```
func wordPattern(pattern string, str string) bool {
    singleStrs := strings.Split(str, " ")
    if len(pattern) != len(singleStrs) {
        return false
    }
    for i := 0; i < len(pattern); i++ {
        singleP := pattern[i]
        singleStr := singleStrs[i]
        for j := 0; j < len(pattern); j++ {
            if i == j {
                continue
            }
            _singleP := pattern[j]
            _singleStr := singleStrs[j]
            if singleP == _singleP && singleStr != _singleStr {
                return false
            }
            switch {
            case singleP == _singleP && singleStr != _singleStr: 
                return false
            case singleP != _singleP && singleStr == _singleStr:
                return false
            }
        } 
    }
    return true
}
```
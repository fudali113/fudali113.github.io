title: go 1.6.2 strings split 方法改造
date: 2016-12-13
tags: ["golang"]
---

当调用`strings.Split(s,seq string)`时,如果seq连续出现，比如`s=" dfdgdfg              （多个空格）        dfdg   （多个空格）   hghyjkjuyk      "`。调用`slice:=strings.Split(s," ")`将会出现`len(slice)!=3`，我认为这并不是大家希望看到的结果。

查看`strings.Split(s,seq string)`源码：
`func Split(s, sep string) []string { return genSplit(s, sep, 0, -1) }`

接着查看`strings.genSplit()`源码：
```
  func genSplit(s, sep string, sepSave, n int) []string {
			if n == 0 {
				return nil
			}
			if sep == "" {
				return explode(s, n)
			}
			if n < 0 {
				n = Count(s, sep) + 1
			}
			c := sep[0]
			start := 0
			a := make([]string, n)
			na := 0
			for i := 0; i+len(sep) <= len(s) && na+1 < n; i++ {
				if s[i] == c && (len(sep) == 1 || s[i:i+len(sep)] == sep) {
					a[na] = s[start : i+sepSave]
					na++
					start = i + len(sep)
					i += len(sep) - 1
				}
			}
			a[na] = s[start:]
			return a[0 : na+1]
		}
```

发现并没有做相关的判断就将`s[start : i+sepSave]`添加到返回数组造成出现这种情况；

顾在for循环中添加一个判断以达到预期返回值，代码如下：
```
if s[i] == c && (len(sep) == 1 || s[i:i+len(sep)] == sep) {
			splitStr:=s[start : i+sepSave]
			if !(splitStr == sep || start==i+sepSave) {
				a[na] = splitStr
				na++
			}
			start = i + len(sep)
			i += len(sep) - 1
}
```

之后调用即可达到预期返回值

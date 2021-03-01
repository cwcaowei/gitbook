#### 滑动窗口

- [LeetCode第76题](https://leetcode-cn.com/problems/odd-even-linked-list/)
```java
public String minWindow(String s, String t) {
    // need记录目标字符串t中各字符的个数
    Map<Character, Integer> need = new HashMap();
    for (char c : t.toCharArray()) {
        need.put(c, need.getOrDefault(c, 0) + 1);
    }

    // window记录滑动过程中窗口内各字符的个数
    Map<Character, Integer> window = new HashMap();
    // 记录window中有多少字符的个数已达到need中对应字符的个数
    int valid = 0,
    // 记录最小子串的起始位置
    int start = 0;
    // 记录最小子串的长度
    int resultLength = Integer.MAX_VALUE;
    int left = 0, right = 0,

    while (right < s.length()) {
        // 判断字符是否是need中的
        char rc = s.charAt(right);
        if (need.containsKey(rc)) {
            // 如果是，更新window中该字符的个数
            window.put(rc, window.getOrDefault(rc, 0) + 1);
            // 如果window中该字符个数已达到need中该字符个数，更新valid
            if (window.get(rc).intValue() == need.get(rc).intValue()) {
                valid++;
            }
        }
        // 扩大窗口
        right++;
        // window内的字符及个数已满足need中的各字符及个数，这时要开始缩小窗口
        while (valid == need.size()) {
            // 更新最小子串的长度和起始位置
            if (right - left < resultLength) {
                resultLength = right - left;
                start = left;
            }
            // 获取缩小时移出窗口的字符
            char lc = s.charAt(left);
            // 判断字符是否是need中的
            if (need.containsKey(lc)) {
                // 如果是，更新window中该字符的个数
                window.put(lc, window.get(lc) - 1);
                // 更新后，如果window中该字符个数正好比need中该字符个数少1，更新valid
                if (window.get(lc).intValue() + 1 == need.get(lc).intValue()) {
                    valid--;
                }
            }
            // 缩小窗口
            left++;
        }
    }

    return resultLength == Integer.MAX_VALUE ? "" : s.substring(start, start + resultLength);
}
```

- [LeetCode第567题](https://leetcode-cn.com/problems/permutation-in-string/)
```java
public boolean checkInclusion(String s1, String s2) {
    Map<Character, Integer> need = new HashMap();
    for (char c : s1.toCharArray()) {
        need.put(c, need.getOrDefault(c, 0) + 1);
    }
    Map<Character, Integer> window = new HashMap();
    int left = 0;
    int right = 0;
    int valid = 0;
    int length = 0;
    while (right < s2.length()) {
        char rc = s2.charAt(right);
        if (need.containsKey(rc)) {
            window.put(rc, window.getOrDefault(rc, 0) + 1);
            if (window.get(rc).intValue() == need.get(rc).intValue()) {
                valid++;
            }
        }
        right++;
        // 每次扩大或缩小窗口后要重新计算窗口长度length
        length = right - left;
        while (valid == need.size()) {
            // 如果窗口长度正好是s1长度，则满足条件了
            // 否则窗口中包含s1中没有的字符，要缩小窗口来去掉这些字符
            if (length == s1.length()) {
                return true;
            }
            char lc = s2.charAt(left);
            left++;
            if (need.containsKey(lc)) {
                window.put(lc, window.get(lc) - 1);
                if (window.get(lc).intValue() == need.get(lc).intValue() - 1) {
                    valid--;
                }
            }
            // 每次扩大或缩小窗口后要重新计算length
            length = right - left;
        }
    }
    return false;
}
````

- [LeetCode第438题](https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/)
```java
public List<Integer> findAnagrams(String s, String p) {
    List<Integer> list = new ArrayList();
    Map<Character, Integer> need = new HashMap();
    for (char c : p.toCharArray()) {
        need.put(c, need.getOrDefault(c, 0) + 1);
    }
    Map<Character, Integer> window = new HashMap();
    int left = 0;
    int right = 0;
    int valid = 0;
    int length = 0;
    while (right < s.length()) {
        char rc = s.charAt(right);
        if (need.containsKey(rc)) {
            window.put(rc, window.getOrDefault(rc, 0) + 1);
            if (window.get(rc).intValue() == need.get(rc).intValue()) {
                valid++;
            }
        }
        right++;
        // 每次扩大或缩小窗口后要重新计算窗口长度length
        length = right - left;
        while (valid == need.size()) {
            // 如果窗口长度正好是s长度，则满足条件了,记录至list
            // 因为此时valid没有减少，所以要显式结束循环
            // 否则窗口中包含s中没有的字符，要缩小窗口来去掉这些字符
            if (length == p.length()) {
                list.add(left);
                break;
            } else {
                char lc = s.charAt(left);
                left++;
                if (need.containsKey(lc)) {
                    window.put(lc, window.get(lc) - 1);
                    if (window.get(lc).intValue() == need.get(lc).intValue() - 1) {
                        valid--;
                    }
                }
                // 每次扩大或缩小窗口后要重新计算length
                length = right - left;
            }
        }
    }
    return list;
}
```

- [LeetCode第3题](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)
```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> window = new HashMap();
    int left = 0;
    int right = 0;
    int length = 0;
    while (right < s.length()) {
        char rc = s.charAt(right);
        window.put(rc, window.getOrDefault(rc, 0) + 1);
        // 扩大窗口
        right++;
        // 当窗口内存在重复字符时要开始缩小窗口直至没有重复字符
        while (window.get(rc) > 1) {
            char lc = s.charAt(left);
            // 缩小窗口
            left++;
            // 移除窗口最左边的字符
            window.put(lc, window.get(lc) - 1);
        }    
        // 比较此时的窗口长度与目前满足题意的最长子串
        length = Math.max(length, right - left);
    }
    return length;
}
```

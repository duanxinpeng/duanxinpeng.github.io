## 问题描述
`给定一个主字符串 text 和一个模式字符串 pattern，返回字符串 text 中和字符串 pattern 相同的子串的起始位置`。
## 实现方法
### BR暴力法
1. 先找到和 s 中和 p 的第一个字符相等的字符的位置
2. 从步骤 1 返回的位置开始，判断 s 中从此处开始的子串能否与 p 完全匹配
3. 步骤 2 的判断方法是逐个字符进行比较，采用java 的 equals() 方法并不能优化时间复杂度，因为 equals 的底层实现依然是逐字符比较。
3. java 的 indexOf 采用的就是这种方法。
4. 时间复杂度O(nm)(n和m分别为匹配（长）串、被匹配（短）串的长度)
### RK算法
1. RK 算法是对BR暴力实现的一种优化，优化的方式是加快模式串的比较，而不是加快模式串的移动！
2. 通过为模式串以及主串中和模式串等长的子串生成 hashcode，进而通过比较 hashcode 来确定两个字符串是否相等。如果不相等再进行逐字符的比较。
3. 在此算法中，最重要的就是哈希函数的选择
	- 必须采用滚动哈希，从而将每次哈希值的生成的复杂度将为1.
	- 主要利用了 Rabin fingerprint 的思想。
	- 计算字符串 $t[0,m-1]$ 的哈希值的公式：$Hash(t[0,m-1]) = t[0]*b^{m-1} + t[1]*b^{m-2}+...+t[m-1]*b^0 (t[0]表示该字符的ASCII码,b是一个常数，取值256)$
	- 那么计算 t[1,m] 的哈希值公式为：$Hash(t[1,m]) = (Hash(t[0,m-1])-t[0]*b^{m-1})*b + t[m]*b^0$
	- 同时为了防止溢出，还需要对其取模，这个值应该是尽可能大的质数，可以取 101.
4. 代码
	```
    //Rabin-Karp
    final int BASE = 256;
    final int MODULES = 101;

    public int rabinKarp(String text, String pattern) {
        int tLen = text.length();
        int pLen = pattern.length();
        int tHash = 0;
        int pHash = 0;
        for (int i = 0; i < pLen; i++) {
            tHash = (BASE * tHash + text.charAt(i)) % MODULES;
            pHash = (BASE * pHash + pattern.charAt(i)) % MODULES;
        }
        // 作滚动哈希用
        int h = 1;
        for (int i = 0; i < pLen - 1; i++) {
            h = (h * BASE) % MODULES;
        }
        int i = 0;
        while (i <= tLen - pLen) {
            //先通过哈希值比较，再通过 equals 方法进行比较
            if (tHash == pHash && pattern.equals(text.substring(i, i + pLen)))
                return i;
            //比较失败，生成下一个哈希值
            if (i < tLen - pLen)
                tHash = (BASE * (tHash + MODULES - text.charAt(i) * h ) + text.charAt(i + pLen)) % MODULES;

            //防止出现负数？？？？？
            if (tHash < 0) {
                tHash = tHash + MODULES;
            }
            i++;
        }
        return -1;
    }
	```
5. reference
	- https://en.jinzhao.wiki/wiki/Rabin%E2%80%93Karp_algorithm
	- https://ethsonliu.com/2019/12/rabin-karp.html

### BM 算法
1. 模式串的比较`从右到左`，模式串的移动`从左到右`！
2. 是对`后缀匹配`暴力算法的改进。
3. 为了实现更快移动模式串，定义了`坏字符`和`好后缀`两个规则，移动模式串时不再是简单的 $j++$， 而是 $j+max(shift(好后缀),shitf(坏字符))$。
	- 坏字符
		- 从后向前比，主串中第一个和模式串不匹配的字符被称为坏字符。
		- 找到坏字符之后，将模式串向后移动，直到模式串中与坏字符相同的字符移动到与坏字符相同的位置为止
		- 如果模式串中没有和坏字符相同的字符，就会将模式串整个移动到坏字符之后。
	- 好后缀
		- 好后缀指的是模式串和子串当中相匹配的后缀。
		- 如果模式串中包含好后缀，就将模式串中的好后缀与主串中的好后缀对齐
		- 如果没有好后缀，则需要确定既是好后缀又是patter的前缀的最长子串的长度
		- 如果以上两种情况都没有，直接将pattern向后移动pattern.length。
4. 代码
	```
	public void preBc(int[] bmBc, String pattern) {
        int m = pattern.length();
        int num = bmBc.length;
        for (int i = 0; i < num; i++) {
            bmBc[i] = m;
        }
        for (int i = 0; i < m; i++) {
            bmBc[pattern.charAt(i)] = m - 1 - i;
        }
    }

    public void suffiOld(String pattern, int[] suff) {
        int m = pattern.length();
        suff[m - 1] = m;
        for (int i = m - 2; i >= 0; i--) {
            int j = i;
            while (j >= 0 && pattern.charAt(j) == pattern.charAt(m - 1 - i + j))
                j--;
            suff[i] = i - j;
        }
    }

    public void suffix(String pattern, int[] suff) {
        int m = pattern.length();
        suff[m - 1] = m;
        int g = m - 1, f = 0;
        for (int i = m - 2; i >= 0; i--) {
            if (i > g && suff[i + m - 1 - f] < i - g) {
                suff[i] = suff[i + m - 1 - f];
            } else {
                if (i < g)
                    g = i;
                f = i;
                while (g >= 0 && pattern.charAt(g) == pattern.charAt(g + m - 1 - f))
                    g--;
                suff[i] = f - g;
            }
        }
    }

    public void preBmGs(String pattern, int[] bmGs) {
        int m = pattern.length();
        int[] suff = new int[m];
        suffix(pattern, suff);
        //case 3
        for (int i = 0; i < m; i++) {
            bmGs[i] = m;
        }
        //case 2
        for (int i = m - 1; i >= 0; i--) {
            if (suff[i] == i + 1) {
                for (int j = 0; j < m - 1; j++) {
                    if (bmGs[j] == m)
                        bmGs[j] = m - 1 - j;
                }
            }
        }
        //case 1
        for (int i = 0; i <= m - 2; i++) {
            bmGs[m - 1 - suff[i]] = m - 1 - i;
        }
    }

    public int BoyerMoore(String pattern, String text) {
        int m = pattern.length();
        int n = text.length();
        int[] bmBc = new int[256];
        int[] bmGs = new int[m];
        preBc(bmBc, pattern);
        preBmGs(pattern, bmGs);
        int j = 0;
        while (j <= n - m) {
            int i;
            for (i = m - 1; i >= 0 && pattern.charAt(i) == text.charAt(i + j); i--) ;
            if (i < 0) {
                return j;
            } else {
                j += Math.max(bmBc[text.charAt(i + j)] - m + 1 + i, bmGs[i]);
            }
        }
        return -1;
    }
	```
5. reference
	- https://www.cnblogs.com/lanxuezaipiao/p/3452579.html
	- http://www-igm.univ-mlv.fr/~lecroq/string/node14.html
	- http://www.cs.utexas.edu/users/moore/publications/fstrpos.pdf
	- https://www.cnblogs.com/wxgblogs/p/5701101.html

### KMP 算法
1. 通过更快移动模式串来优化 BR 算法。
2. next[i]表示pattern[i]表示的最长后缀相等的最长前缀的最后一个位置的下标，也就是说通过next数组，可以直接得到对应的最长前缀的位置。
3. 之所以可以用`j=next[j]`这种写法，是因为pattern[i-1]和pattern[j]是一样的。
3. 代码
```java
    //KMP算法
	// next数组的构建与kmp算法结构上很相似
    public static void preNext(int[] next, String p) {
        int m = p.length();
        for (int i = 0; i < m; i++) {
            next[i] = -1;
        }
        for (int i = 1; i < m; i++) {
            int j = next[i - 1];
            while (j != -1 && p.charAt(j + 1) != p.charAt(i))
                j = next[j];
            if (p.charAt(j + 1) == p.charAt(i))
                next[i] = j + 1;
        }
    }

    public static int KMP(String text, String pattern) {
        int pLen = pattern.length();
        int tLen = text.length();
        int[] next = new int[pLen];
        preNext(next, pattern);
        // 被匹配串pattern的index
        int match = -1;
		// i 不会后退！
        for (int i = 0; i < tLen; i++) {
			// 只会让match后退，而且后退多少由next数组决定
            while (match != -1 && pattern.charAt(match + 1) != text.charAt(i))
                match = next[match];
            if (pattern.charAt(match + 1) == text.charAt(i)) {
                match++;
                if (match == pLen - 1)
                    return (i-match);
            }
        }
        return -1;
    }
```
2. 时间复杂度 $O(n+m)$
	- next数组构建复杂度：
	- 比较时间复杂度
2. reference
	- https://ethsonliu.com/2018/04/kmp.html
	- http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html
### Sunday 算法
1. reference
	- https://ethsonliu.com/2019/08/sunday-algorithm.html
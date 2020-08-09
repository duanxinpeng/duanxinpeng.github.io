## 问题描述
1. https://leetcode-cn.com/problems/trapping-rain-water/
## Solutions
1. 暴力法：遍历数组中的每一个元素，并分别向左、向右寻找比当前元素大的最大元素，两者取其小，减掉当前元素就是当前元素上面可存储的水量；将所有元素上面可存储的水量加起来就是最后的结果。
	```
	int res = 0;
	int n = height.length;
	for (int i = 0; i < n; i++) {
		int left = height[i];
		for (int j = 0; j < i; j++) {
			if (height[j] > left)
				left = height[j];
		}
		int right = height[i];
		for (int j = i+1; j < n; j++) {
			if (height[j] > right) {
				right = height[j];
			}
		}
		res += (Math.max(left,right)-height[i]);
	}
	```
2. 动态编程：用两个数组，分别将元素 i 左边和右边的最大元素存储起来即可。记得判断当前数组是否为空。
	```
	int n = height.length;
	if (n == 0)
		return 0;
	int[] left = new int[n];
	int[] right = new int[n];
	int max = height[0];
	for (int i = 0; i < n; i++) {
		if (height[i] > max)
			max = height[i];
		left[i] = max;
	}
	max = height[n-1];
	for (int i = n-1; i >=0 ; i--) {
		if (height[i] > max) {
			max = height[i];
		}
		right[i] = max;
	}
	int res = 0;
	for (int i = 0; i < n; i++) {
		res += (Math.min(left[i],right[i])-height[i]);
	}
	```
3. 单调栈：单调递减栈
	```
	int n = height.length;
	if (n == 0) {
		return 0;
	}
	int res = 0;
	Deque<Integer> queue = new ArrayDeque<>();
	for (int i = 0; i < n; i++) {
		while (!queue.isEmpty() && height[i] > height[queue.peek()]) {
			int top = queue.poll();
			if (queue.isEmpty())
				break;
			res += (i-queue.peek()-1)*(Math.min(height[i],height[queue.peek()]));
		}
		queue.offer(i);
	}
	```
4. 双指针：用两个指针分别记录左右两个方向的最大值，我们只需要考虑最大值更小的那一个方向即可！
	```
	int n = height.length;
	if (n == 0)
		return 0;
	int left = 0, right = n-1;
	int i = 0, j = n-1;
	int res = 0;
	while (i < j) {
		if (height[i] < height[j]) {
			if (height[i] > height[left])
				left = i;
			res = res + height[left] - height[i];
			i++;
		} else {
			if (height[j] > height[right])
				right = j;
			res = res + height[right] - height[j];
			j--;
		}
	}
	```
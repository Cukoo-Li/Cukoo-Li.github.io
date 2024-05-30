---
title: "快速排序和归并排序"
date: 2024-04-30
---

快速排序和归并排序是面试中常考的排序算法。

本文首先简单介绍快速排序和归并排序的算法思想，然后分别给出两者在数组和链表两种数据结构中的算法实现（迭代法+递归法）。

## 快速排序

快速排序的算法思想是：在待排序表`{1...n}`中任取一个元素作为枢轴`pivot`，通过一趟排序将待排序表划分为独立的两部分`{1...k-1}`和`{k+1...n}`，使得`{1...k-1}`中的所有元素小于枢轴`pivot`，`{k+1...n}`中所有元素大于等于枢轴`pivot`，则枢轴`pivot`放在了其最终位置`k`上，这个过程称为一趟快速排序（或一次划分）。然后分别递归地对两个子表重复上述过程，直至子表内只有一个元素或为空为止，即所有元素都放在了其最终位置上。

由此可见，快速排序本身就是一种基于递归的算法。算法实现的关键是要在单次递归逻辑中，将给定的待排序表划分成两部分。此外，我们还可以使用栈去模拟递归调用的过程。

### 排序数组

- 递归法

  ```cpp
  // 对区间[left, right]内的元素进行快速排序
  void QuickSort(vector<int>& nums, int left, int right) {
      if (left >= right)
          return;
      int pivot_idx = Partition(nums, left, right);       // 进行一次划分
      QuickSort(nums, left, pivot_idx - 1);       // 对左半部分进行快速排序
      QuickSort(nums, pivot_idx + 1, right);      // 对右半部分进行快速排序
  }
  
  // 对区间[left, right]内的元素进行划分，返回枢轴下标
  int Partition(vector<int>& nums, int left, int right) {
      int pivot = nums[left]; // 选取首元素作为枢轴
      while (left < right) {
          // 从右往左扫描，将小于枢轴的元素放到左边
          while (left < right && nums[right] >= pivot)
              --right;
          nums[left] = nums[right];
          // 从左往右扫描，将大于等于枢轴的元素放到右边
          while (left < right && nums[left] < pivot)
              ++left;
          nums[right] = nums[left];
      }
      nums[left] = pivot;     // 将枢轴放到最终位置上
      return left;
  }
  ```

- 迭代法

  ```cpp
  // 对区间[left, right]内的元素进行快速排序
  void QuickSort(vector<int>& nums, int left, int right) {
      if (left >= right)
          return;
      stack<pair<int, int>> sta;      // 用栈存储区间范围，用于调用Partition
      sta.push({left, right});
      while (!sta.empty()) {
          // 获取当前区间范围
          auto interval = sta.top();
          sta.pop();
          int pivot_idx = Partition(nums, interval.first, interval.second);
          // 将左半部分区间范围入栈
          if (interval.first < pivot_idx - 1)
              sta.push({interval.first, pivot_idx - 1});
          // 将右半部分区间范围入栈
          if (pivot_idx + 1 < interval.second)
              sta.push({pivot_idx + 1,interval.second});
      }
  }
  
  // 对区间[left, right]内的元素进行划分，返回枢轴下标
  int Partition(vector<int>& nums, int left, int right) {
      int pivot = nums[left]; // 选取首元素作为枢轴
      while (left < right) {
          // 从右往左扫描，将小于枢轴的元素放到左边
          while (left < right && nums[right] >= pivot)
              --right;
          nums[left] = nums[right];
          // 从左往右扫描，将大于等于枢轴的元素放到右边
          while (left < right && nums[left] < pivot)
              ++left;
          nums[right] = nums[left];
      }
      nums[left] = pivot;     // 将枢轴放到最终位置上
      return left;
  }
  ```

### 排序链表

- 递归法

  ```cpp
  class Solution {
      // 对区间(dummy_head, end)内的元素进行快速排序
      void QuickSort(ListNode* dummy_head, ListNode* end) {
          if (dummy_head->next == end || dummy_head->next->next == end)
              return;
          ListNode* pivot = Partition(dummy_head, end); // 划分，确定枢轴
          QuickSort(dummy_head, pivot); // 对左半部分进行快速排序
          QuickSort(pivot, end);        // 对右半部分进行快速排序
      }
  
      // 对区间(dummy_head, end)内的元素进行划分，返回指向枢轴的指针
      ListNode* Partition(ListNode* dummy_head, ListNode* end) {
          ListNode* pivot = dummy_head->next; // 选取首元素作为枢轴元素
          ListNode* prev = pivot;
          // 遍历链表，将小于枢轴的元素插入到链表头
          while (prev->next != end) {
              if (prev->next->val < pivot->val) {
                  ListNode* node = prev->next;
                  prev->next = prev->next->next;
                  ListNode* tmp = dummy_head->next;
                  dummy_head->next = node;
                  node->next = tmp;
              } else
                  prev = prev->next;
          }
          return pivot;
      }
  
  public:
      ListNode* sortList(ListNode* head) {
          ListNode* dummy_head = new ListNode(0, head);
          QuickSort(dummy_head, nullptr);
          head = dummy_head->next;
          delete dummy_head;
          return head;
      }
  };
  ```

- 迭代法

  ```cpp
  class Solution {
      // 对区间(dummy_head, end)内的元素进行快速排序
      void QuickSort(ListNode* dummy_head, ListNode* end) {
          if (dummy_head->next == end || dummy_head->next->next == end)
              return;
          stack<pair<ListNode*, ListNode*>> sta; // 用栈存储区间范围，用于调用Partition
          sta.push({dummy_head, end});
          while (!sta.empty()) {
              // 获取当前区间范围
              auto interval = sta.top();
              sta.pop();
              ListNode* pivot = Partition(interval.first, interval.second);
              // 将左半部分区间范围入栈
              if (interval.first->next != pivot &&
                  interval.first->next->next != pivot)
                  sta.push({interval.first, pivot});
              // 将右半部分区间范围入栈
              if (pivot->next != interval.second &&
                  pivot->next->next != interval.second)
                  sta.push({pivot, interval.second});
          }
      }
  
      // 对区间(dummy_head, end)内的元素进行划分，返回指向枢轴的指针
      ListNode* Partition(ListNode* dummy_head, ListNode* end) {
          ListNode* pivot = dummy_head->next; // 选取首元素作为枢轴元素
          ListNode* prev = pivot;
          // 遍历链表，将小于枢轴的元素插入到链表头
          while (prev->next != end) {
              if (prev->next->val < pivot->val) {
                  ListNode* node = prev->next;
                  prev->next = prev->next->next;
                  ListNode* tmp = dummy_head->next;
                  dummy_head->next = node;
                  node->next = tmp;
              } else
                  prev = prev->next;
          }
          return pivot;
      }
  
  public:
      ListNode* sortList(ListNode* head) {
          ListNode* dummy_head = new ListNode(0, head);
          QuickSort(dummy_head, nullptr);
          head = dummy_head->next;
          delete dummy_head;
          return head;
      }
  };
  ```

## 归并排序

归并排序的思想是：初始时，将待排序表视为`n`个只含`1`个元素的有序子表，然后两两归并，合并成一个更大的有序表，直到合并成一个长度为`n`的有序表为止。

由此可见，归并排序的思想非常简洁，关键是一开始要找出只含`1`个元素的有序子表，然后慢慢生成更长的有序表。这个过程可以使用递归法自顶向下地实现，也可以使用迭代法自底向上地实现。

### 排序数组

- 递归法

  ```cpp
  vector<int> tmp;
  
  // 对区间[left, right]内的元素进行归并排序
  void MergeSort(vector<int>& nums, int left, int right) {
      if (left >= right)
          return;
      int mid = left + (right - left) / 2;  // 从中间划分
      MergeSort(nums, left, mid);           // 对左半部分进行归并排序
      MergeSort(nums, mid + 1, right);      // 对右半部分进行归并排序
      Merge(nums, left, mid, right);  // 左半部分和右半部分各自有序，将二者归并
  }
  
  // 区间[left, mid]和区间[mid + 1, right]各自有序，将二者归并
  void Merge(vector<int>& nums, int left, int mid, int right) {
      // 对区间内的元素进行备份
      tmp.resize(nums.size(), 0);
      std::copy(nums.begin() + left, nums.begin() + right + 1,
                tmp.begin() + left);
      // 三指针，依次取较小值
      int i = left;
      int j = mid + 1;
      int k = left;
      while (i <= mid && j <= right) {
          if (tmp[i] <= tmp[j])
              nums[k++] = tmp[i++];
          else
              nums[k++] = tmp[j++];
      }
      // 将较长有序表中剩余的元素依次取完
      while (i <= mid)
          nums[k++] = tmp[i++];
      while (j <= right)
          nums[k++] = tmp[j++];
  }
  ```

- 迭代法

  ```cpp
  vector<int> tmp;
  
  // 对区间[left, right]内的元素进行归并排序
  void MergeSort(vector<int>& nums) {
      if (left >= right)
          return;
      // 从步长d为1开始归并，每趟翻倍
      for (int d = 1; d < nums.size(); d *= 2) {
          int cur = 0;  // 每次归并的起点
          while (cur < nums.size()) {
              // 确定将要归并的两个区间边界
              int left = cur;
              int mid = cur + d - 1;
              int right = std::min(static_cast<int>(nums.size() - 1), mid + d);
              if (mid < right)
                  Merge(nums, left, mid, right);
              cur = right + 1;
          }
      }
  }
  
  // [left, mid]和[mid + 1, right]各自有序，将二者归并
  void Merge(vector<int>& nums, int left, int mid, int right) {
      // 对区间内的元素进行备份
      tmp.resize(nums.size(), 0);
      std::copy(nums.begin() + left, nums.begin() + right + 1,
                tmp.begin() + left);
      // 三指针，依次取较小值
      int i = left;
      int j = mid + 1;
      int k = left;
      while (i <= mid && j <= right) {
          if (tmp[i] <= tmp[j])
              nums[k++] = tmp[i++];
          else
              nums[k++] = tmp[j++];
      }
      // 将较长有序表中剩余的元素依次取完
      while (i <= mid)
          nums[k++] = tmp[i++];
      while (j <= right)
          nums[k++] = tmp[j++];
  }
  ```

### 排序链表

- 递归法

  ```cpp
  class Solution {
      // 对链表head进行归并排序
      ListNode* MergeSort(ListNode* head) {
          if (!head || !head->next)
              return head;
          // 将链表对半切开
          ListNode* slow = head;
          ListNode* fast = head;
          while (fast->next && fast->next->next) {
              slow = slow->next;
              fast = fast->next->next;
          }
          ListNode* left = head;
          ListNode* right = slow->next;
          slow->next = nullptr;
          // 对左半部分和右半部分分别进行归并排序
          left = MergeSort(left);
          right = MergeSort(right);
          // 归并
          return Merge(left, right);
      }
  
      // 链表left和right各自有序，将二者归并
      ListNode* Merge(ListNode* left, ListNode* right) {
          ListNode* dummy_head = new ListNode(0);
          ListNode* cur = dummy_head;
          while (left && right) {
              if (left->val <= right->val) {
                  cur->next = left;
                  left = left->next;
              } else {
                  cur->next = right;
                  right = right->next;
              }
              cur = cur->next;
          }
          cur->next = left ? left : right;
          ListNode* head = dummy_head->next;
          delete dummy_head;
          return head;
      }
  public:
      ListNode* sortList(ListNode* head) {
          return MergeSort(head);
      }
  };
  ```

- 迭代法

  ```cpp
  class Solution {
      // 对链表head进行归并排序
      ListNode* MergeSort(ListNode* head) {
          if (!head || !head->next)
              return head;
          // 计算链表长度
          int length = 0;
          for (ListNode* cur = head; cur; cur = cur->next)
              ++length;
          // 创建虚拟头结点
          ListNode* dummy_head = new ListNode(0, head);
          // 从步长d为1开始归并，每趟翻倍
          for (int d = 1; d < length; d *= 2) {
              ListNode* cur = dummy_head->next;
              ListNode* prev = dummy_head; // 用于连接归并结果
              // 按步长d提取出左、右子链表
              while (cur) {
                  ListNode* left = cur;
                  for (int i = 1; i < d && cur->next; ++i)
                      cur = cur->next;
                  ListNode* right = cur->next;
                  // 如果子链表right不存在，不用归并，直接连接子链表left
                  if (!right) {
                      prev->next = left;
                      break;
                  }
                  cur->next = nullptr;
                  cur = right;
                  for (int i = 1; i < d && cur->next; ++i)
                      cur = cur->next;
                  // 保存剩余的未归并的链表
                  ListNode* remain = cur->next;
                  cur->next = nullptr;
                  // 归并，并连接结果
                  prev->next = Merge(left, right);
                  // 更新prev和cur
                  while (prev->next)
                      prev = prev->next;
                  cur = remain;
              }
          }
          head = dummy_head->next;
          delete dummy_head;
          return head;
      }
      // 链表left和right各自有序，将二者归并
      ListNode* Merge(ListNode* left, ListNode* right) {
          ListNode* dummy_head = new ListNode(0);
          ListNode* cur = dummy_head;
          while (left && right) {
              if (left->val <= right->val) {
                  cur->next = left;
                  left = left->next;
              } else {
                  cur->next = right;
                  right = right->next;
              }
              cur = cur->next;
          }
          cur->next = left ? left : right;
          ListNode* head = dummy_head->next;
          delete dummy_head;
          return head;
      }
  
  public:
      ListNode* sortList(ListNode* head) { 
          return MergeSort(head); 
      }
  };
  ```
  
  

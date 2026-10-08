```toc
```
## 概要

权限系统的演化。

权限系统，要解决的问题，是一个用户，在登录后，拥有那些权限。

这涉及到如下几个小问题

1. 如何定义权限
2. 如何授权
3. 如何鉴权

```python
def partition(arr,low,high): 
    i = ( low-1 )         # 最小元素索引
    pivot = arr[high]     
  
    for j in range(low , high): 
  
        # 当前元素小于或等于 pivot 
        if   arr[j] <= pivot: 
          
            i = i+1 
            arr[i],arr[j] = arr[j],arr[i] 
  
    arr[i+1],arr[high] = arr[high],arr[i+1] 
    return ( i+1 ) 
  
 
# arr[] --> 排序数组
# low  --> 起始索引
# high  --> 结束索引
  
# 快速排序函数
def quickSort(arr,low,high): 
    if low < high: 
  
        pi = partition(arr,low,high) 
  
        quickSort(arr, low, pi-1) 
        quickSort(arr, pi+1, high) 
  
arr = [10, 7, 8, 9, 1, 5] 
n = len(arr) 
quickSort(arr,0,n-1) 
print ("排序后的数组:") 
for i in range(n): 
    print ("%d" %arr[i]),
```

## 访问控制模型-主谓宾

许多访问控制模型 Access Control，本质上都是主谓宾的结构：
1. 定义哪些资源可以访问
2. 定义谁可以访问
3. 定义怎么访问


|            | Subject                       | Object                                   | Verb              |
| ---------- | ----------------------------- | ---------------------------------------- | ----------------- |
| Unix安全模型   | process, user                 | file, dir, socket...                     | read, write, exec |
| 浏览器安全模型    | origin (scheme://domain:port) | cookie, DOM, js namespace, HW permission | access            |
| cookie安全模型 | origin (domain + path)        | cookie                                   | access            |

## UNIX安全模型-ACLs

权限模型好像都是主谓宾结构

Subjects (Who?) ：Users, processes 
Objects (What?) 
- Files, directories 
- Files: sockets, pipes, hardware devices, kernel objects, process data Access 
Operations (How?) ：Read, Write, Execute

Access Control List. The most common method of implementing access control in a computer system is through access control lists.All system resources, such as files, printers, and terminals, have a list of authorized users attached.

```
-rw-r--r--@    1 wanshuo  staff        3515  7 30 14:29 jbr_err_pid94423.log
drwxr-xr-x     5 wanshuo  staff         160  4 18  2022 logs/
-rw-r-----     1 wanshuo  staff     8053492  8 15 16:32 mmi.tar.gz
drwxr-xr-x     5 wanshuo  staff         160  4 18 14:41 nacos/
drwxr-xr-x     5 wanshuo  staff         160  4 13 09:30 nltk_data/
drwxr-xr-x    13 wanshuo  staff         416  4  3 15:32 pic/
drwxr-xr-x     6 wanshuo  staff         192  1 30  2023 projects/
drwxr-xr-x     3 wanshuo  staff          96  3 22  2022 reading/
```

![](../drawing/acls.excalidraw)

给用户授权的整体复杂度：$\lvert N \rvert$
增加一个权限(文件)的复杂度： $\lvert U \rvert$
查找一个用户有哪些权限：  $\lvert N * U \rvert$
查找一个权限(文件)有哪些用户拥有： $\lvert U \rvert$
删除一个权限(文件)的复杂度：$1$
删除一个用户的复杂度： $\lvert N*U \rvert$

理想中的 ACLs 受制于 Object 的数量，当系统中存在大量的文件时，查找用户拥有的权限，是极其复杂的。它是面向 Object 的数据模型。

改良后的 ACLs （list 中存放的是用户组）

![](../drawing/acls_group.excalidraw)

给用户授权的整体复杂度： $\lvert U\_Group \rvert * \lvert N \rvert$
增加一个权限(文件)的复杂度： $\lvert U\_Group \rvert$
查找一个用户有哪些权限：  $\lvert N \rvert * \lvert U\_Group \rvert$
查找一个权限(文件)有哪些用户拥有： $\lvert U\_Group  \rvert * \lvert U \rvert$
删除一个权限(文件)的复杂度：$1$
删除一个用户的复杂度： $\lvert U\_Group \rvert$

总结，ACLs 是面向 objects 的权限模型，其复杂度上限受制于 objects 的数量。（RBAC 是面向 Subject 用户的权限模型）

## 笛卡尔积和RBAC

用户如何与权限绑定，最简单的方法，就是做笛卡尔积。

![](../drawing/authorize.excalidraw)

每一个用户，都绑定指定的权限。复杂度是  $\lvert U \rvert * \lvert P \rvert$，每增加一个用户，就要针对这个用户进行授权。$U$ 和 $P$ 是多对多的关系。

给用户授权的整体复杂度： $\lvert U \rvert * \lvert P \rvert$
增加一个权限的复杂度： $\lvert U \rvert$
查找一个用户有哪些权限：  $\lvert P \rvert$
查找一个权限有哪些用户拥有： $\lvert U \rvert * \lvert P \rvert$
删除一个权限的复杂度： $\lvert U \rvert$
删除一个用户的复杂度： $\lvert P \rvert$

----

笛卡尔积的复杂度明显不适合我们的实际应用场景。在企业中，我们通常会把一个公司划分成多个部门，各司其职。同样的，这些部门内部的权限，都可以归成一类。权限归类的好处显而易见，当有新员工加入该部门后，只需要将新员工与归类好的权限组绑定一次即可。$U$ 和 $P\_Group$ 是多对多的关系，$P\_Group$ 和 $P$ 也是多对多的关系，多个部门可以拥有一样的权限，一个部门可以拥有多个权限。

![](../drawing/authorize_group.excalidraw)

对权限进行分组后，我们看一下时间复杂度：

给用户授权的整体复杂度： $\lvert U \rvert * \lvert P\_Group \rvert +   \lvert P\_Group \rvert * \lvert P \rvert =  \lvert P\_Group \rvert  *  \lvert U + P \rvert$
增加一个权限的复杂度： $\lvert P\_Group \rvert$
查找一个用户有哪些权限： $\lvert P\_Group \rvert *  \lvert P \rvert$
查找一个权限有哪些用户拥有： $\lvert P\_Group \rvert *  \lvert U \rvert$
删除一个权限的复杂度： $\lvert P\_Group \rvert$
删除一个用户的复杂度： $\lvert P\_Group \rvert$

复杂度的变化显而易见，由于权限被分组，时间复杂度整体上被降低了。

给用户授权的整体复杂度： $\lvert U \rvert * \lvert P \rvert$ —>  $\lvert P\_Group \rvert *  \lvert U + P \rvert$
增加一个权限的复杂度： $\lvert U \rvert$ —>  $\lvert P\_Group \rvert$
查找一个用户有哪些权限：  $\lvert P \rvert$ —>  $\lvert P\_Group \rvert * \lvert P \rvert$
查找一个权限有哪些用户拥有： $\lvert U \rvert * \lvert P \rvert$ —>  $\lvert P\_Group \rvert * \lvert U \rvert$
删除一个权限的复杂度： $\lvert U \rvert$ —>  $\lvert P\_Group \rvert$
删除一个用户的复杂度： $\lvert P \rvert$ —>  $\lvert P\_Group \rvert$

**对权限进行分组，就是 RBAC 的核心思想。一组权限，代表了一个角色。**

![](../drawing/authorize_group_role.excalidraw)

## 总结

![](./drawing/rbac_compare.pdf)

1. 不管是面向 Object 的 ACLs 还是面向 Subject 的 RBAC，分组后，都能够显著改善复杂度
2. Any problem in computer science can be solved with another layer of indirection. -- David Wheeler


## Reference

- [Role-Based-Access-Control-Second-Edition](https://book.douban.com/subject/2586338/) [download](https://annas-archive.org/md5/b97cb9de7e038025784c6f1d7d9c9f58)
- [Stanford CS155 Lecture 4: Principle of least privilege, access control, and operating systems security](https://cs155.stanford.edu/syllabus.html)

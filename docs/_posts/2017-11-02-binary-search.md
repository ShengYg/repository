---
layout: post
title:  "二分查找 binary search"
date:   2017-11-02 14:00:00 +0800
categories: [algorithms]
tags: []
description: 二分查找 binary search
---

Template:
<center>
<img src="{{ site.baseurl }}/assets/pic/binary_search_template.png" height="400px" >
</center>

~~~cpp
int lower(vector<int>& v, int first, int last, int val){
    int len = last - first;
    while(len > 0){
        int half = len >> 1;
        int mid = first + half;
        if(v[mid] < val){
            first = mid + 1;
            len = len - half - 1;
        }else
            len = half;
    }
    return first;
}

int upper(vector<int>& v, int first, int last, int val){
    int len = last - first;
    while(len > 0){
        int half = len >> 1;
        int mid = first + half;
        if(v[mid] <= val){
            first = mid + 1;
            len = len - half - 1;
        }else{
            len = half;
        }
    }
    return first;
}

int main() {
    vector<int> v{1,1,2,2,3,3};
    cout << lower(v, 0, 6, 2) << endl;
    cout << upper(v, 0, 6, 2) << endl;
}
~~~


---
title: vue 分页组件
tag: 
	- vue
	- 组件
    - 分页
---

## 需求

- 通过配置显示一定数量的页数
- 点击切换当前页码高亮
- 点击切换触发其他操作

<!--more-->

## 代码分解

### props

通过 props 传入 showTotal, 是否显示总页数； 传入 totalPage, 设置总页数；传入 currentPage, 设置初始激活页码； 传入 visiblePage, 设置显示页数

```javascript
props: {
        showTotal: {
            type: Boolean,
            default: false
        },
        currentPage: {
            type: Number,
            default: 1
        },
        totalPage: {
            type: Number,
            default: 25
        },
        visiblePage: {
            type: Number,
            default: 5
        }
    }
```

### data

data 中维护变量 current，用来存放当前激活页码， 初始值使用 props 的 currentPage

```javascript
data() {
        return {
            // 不直接使用currentPage，避免直接修改 父组件数据
            current: this.currentPage
        }
    }
```

### computed

依赖改变时， 通过对 current 和 visiblePage、totalPage等计算， 得到切换时满足显示个数的的页码数组

```javascript
computed: {
    pageNum() {
        // 初始化前后页边界
        let low = 1
        let high = this.totalPage
        let pageArr = []

        // 当总页数超过可见页数时处理
        if (this.totalPage > this.visiblePage) {
            let subVisiblePage = Math.ceil(this.visiblePage / 2)

            // 处理 前几页逻辑
            if (this.current <= subVisiblePage) {
                low = 1
                high = this.visiblePage
            } else if (this.current > subVisiblePage && this.current < this.totalPage - subVisiblePage + 1) {
                // 处理正常切换
                low = this.current - subVisiblePage
                high = this.current + subVisiblePage - 1
            } else {
                // 处理后几页
                low = this.totalPage - this.visiblePage
                high = this.totalPage + this.visiblePage - 1
            }

            // 确定上下页边界，将边界之间页码放入数组
            while(low <= high) {
                pageArr.push(low)
                low++
            }

            return pageArr
        }
    }
}
```

### methods

当页面切换时，当切换的页码设置为页码， 同时将页码通过事件向外派发， 因为是基础组件，不处理业务逻辑

```javascript
methods: {
    pageChange(page) {
        if (this.current !== page) {
            this.current = page
            this.$emit('page', page)
        }
    }
}
```

### template

```html
<template>
    <div class="page-bar">
        <ul>
            <li v-if="current !== 1">
                <a @click="current--"><< 上一页</a>
            </li>
            <li v-for="index in pageNum"  :class="{active: current === index}" :key="index">
                <a @click="pageChange(index)">{{ index }}</a>
            </li>
            <li v-if="current !== totalPage">
                <a @click="current++">下一页 >></a>
            </li>
            <li v-if="showTotal">
                <a>共 <i>{{totalPage}}</i> 页</a>
            </li>
        </ul>
    </div>
</template>
```
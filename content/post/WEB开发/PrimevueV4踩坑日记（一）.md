---
title: "PrimevueV4踩坑日记（一）" # 文章标题
date: 2025-10-17T11:40:00+08:00 # 文章创建日期
draft: false # 是否为草稿，true 表示草稿，不会在网站上显示
author: "runshell" # 文章作者
description: "今日踩坑：Primevue Datatable组件分页问题" # 文章描述
tags: ["Primevue", "vue3", "前端"] # 文章标签
categories: ["WEB开发"] # 文章分类
---

# 坑

Primevue Datatable 组件分页时，无论 totalRecords 设置为多少，分页总页数都为 1。

# 解决方法

如果服务端单次只返回了一页数据，将 totalRecords 设置为该数据的总条数，同时将 lazy 设置为 true。

# 分析过程

Datatable 组件的分页逻辑是根据 totalRecords 和 rows 计算出总页数。如果 lazy 设置为 true，那么总行数为 totalRecords，否则总行数为 data.length，不读取 totalRecords 值。  
参考：[Datatable 组件的源码](https://github.com/primefaces/primevue/blob/master/packages/primevue/src/datatable/DataTable.vue#L2090)

部分代码片段如下：

```vue
<template>
    <DTPaginator
        v-if="paginatorTop"
        :rows="d_rows"
        :first="d_first"
        :totalRecords="totalRecordsLength"
        :pageLinkSize="pageLinkSize"
        :template="paginatorTemplate"
        :rowsPerPageOptions="rowsPerPageOptions"
        :currentPageReportTemplate="currentPageReportTemplate"
        :class="cx('pcPaginator', { position: 'top' })"
        @page="onPage($event)"
        :alwaysShow="alwaysShowPaginator"
        :unstyled="unstyled"
        :data-p-top="true"
        :pt="ptm('pcPaginator')"
    >
</template>

<script>
totalRecordsLength() {
    if (this.lazy) {
        return this.totalRecords;
    } else {
        const data = this.processedData;

        return data ? data.length : 0;
    }
}
</script>
```

# 代码示例

```vue
<script setup>
import { ref, onMounted } from "vue";

const items = ref([]);
const totalRecords = ref(0);
const loading = ref(false);
const rows = ref(10);

// 加载数据
async function loadData(page = 0) {
  loading.value = true;

  try {
    const response = await fetch(
      `/api/data/?page=${page + 1}&page_size=${rows.value}`
    );
    const result = await response.json();

    items.value = result.results;
    totalRecords.value = result.count; // 服务器返回的总记录数
  } catch (error) {
    console.error("加载失败:", error);
  } finally {
    loading.value = false;
  }
}

function onPage(event) {
  loadData(event.page);
}

onMounted(() => {
  loadData(0);
});
</script>

<template>
  <DataTable :value="items" :lazy="true" <!-- 关键：必须启用 lazy -->
    :paginator="true" :rows="rows" :totalRecords="totalRecords"
    :loading="loading" @page="onPage" tableStyle="min-width: 50rem" >
    <Column field="id" header="ID"></Column>
    <Column field="name" header="名称"></Column>
    <Column field="category" header="分类"></Column>
  </DataTable>
</template>
```

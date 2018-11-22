# Android Paging库使用详解



## Paging库概述

![img](http://clmirror.storage.googleapis.com/codelabs/android-paging/img/511a702ae4af43cd.png)

- [**PagedList**](https://developer.android.com/reference/android/arch/paging/PagedList.html) - a collection that loads data in pages, asynchronously. A `PagedList` can be used to load data from sources you define, and present it easily in your UI with a [`RecyclerView`](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html). 用于异步加载数据的集合。`PagedList`可以从数据源加载数据并显示到 [`RecyclerView`]上。
- [**DataSource**](https://developer.android.com/reference/android/arch/paging/DataSource.html) and [**DataSource.Factory**](https://developer.android.com/reference/android/arch/paging/DataSource.Factory.html) -  `DataSource` 是加载`PagedList`数据快照的基类 `DataSource.Factory负责创建 `DataSource`.
- [**LivePagedListBuilder**](https://developer.android.com/reference/android/arch/paging/LivePagedListBuilder.html) - 构建 `LiveData<PagedList>`, 基于 `DataSource.Factory` 和 `PagedList.Config`.
- [**BoundaryCallback**](https://developer.android.com/reference/android/arch/paging/PagedList.BoundaryCallback.html) - signals when a `PagedList` has reached the end of available data.
- [**PagedListAdapter**](https://developer.android.com/reference/android/arch/paging/PagedListAdapter.html) - 是`RecyclerView.Adapter` that presents paged data from `PagedLists` in a `RecyclerView`. `PagedListAdapter` listens to `PagedList` loading callbacks as pages are loaded, and uses `DiffUtil` to compute fine grained updates as new `PagedLists` are received.


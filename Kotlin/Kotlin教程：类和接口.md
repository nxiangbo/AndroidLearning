Kotlin教程：类和接口





## object 关键字

定义：创建一个类，同时创建这个类的实例.

一般用三种用途

- 可以作为单例
- companion object：可以包含工厂方法，或者与该类相关但不需要该类实例调用的方法
- object表达式，可以替代匿名内部类





```kotlin
service.searchRepos(query).enqueue(
            object : Callback<RepoSearchResponse>{
                override fun onFailure(call: Call<RepoSearchResponse>?, t: Throwable?) {
                    Log.d(TAG, "fail to get data")
                    onError(t!!.message ?: "unknown error")
                }

                override fun onResponse(call: Call<RepoSearchResponse>?, response: Response<RepoSearchResponse>?) {
                    Log.d(TAG, "got a response $response")

                    if (response != null) {
                        if (response.isSuccessful){
                            val repos = response.body()?.items?: emptyList()
                            val next = response.body()?.paging?: null
                            Log.d(TAG, next.toString())
                            onSuccess(repos)
                        } else {
                            Log.d(TAG, "fail to get data")
                            onError("unknown error")
                        }
                    }
                }

            })
```




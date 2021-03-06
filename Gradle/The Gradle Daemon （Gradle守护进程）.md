# The Gradle Daemon （Gradle守护进程）



> daemon 是一种在后台执行的计算机程序。

Gradle运行在JVM上，初始化时需要加载几个支持库。因此，启动过程较慢。而Gradle守护进程可以解决此问题。Gradle守护进程是一个长期存在的后台进程，可以比其他情况更快地执行构建。我们通过避免昂贵的引导过程以及利用缓存来实现这一目标，方法是将项目数据保存在内存中。使用守护进程运行Gradle构建与没有使用守护进程完全没有区别。只需配置您是否要使用它 - 其他一切都由Gradle透明处理。 

## Why the Gradle Daemon is important for performance



守护进程是一个长期存在的进程，因此，我们不仅能够避免每次构建项目时JVM的启动成本，而且还能够在内存中缓存有关项目结构，文件，任务等的信息。

原因很简单：通过重用以前版本的计算来提高构建速度。但是，好处是显着的：我们通常会在后续构建中测量构建时间减少15-75％。我们建议您使用--profile分析构建，以了解Gradle Daemon可以为您带来多大的影响。

默认情况下，Gradle守护程序从Gradle 3.0开始默认开启，因此您无需执行任何操作即可从中受益。

如果您在不重复使用任何进程的短暂环境（例如容器）中运行CI构建，则使用守护进程会略微降低性能（由于缓存其他信息）而没有任何好处，并且可能被禁用。
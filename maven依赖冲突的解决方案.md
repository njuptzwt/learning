mvn包依赖冲突的解决方案

通过日志分析到冲突的包

mvn dependeency:tree：查找依赖之间的关系

定位包所在位置：使用<exclusions></exclusions>来排除相应的依赖
# npn 和 yarn 的网络问题

`.npmrc`:

``` .npmrc
proxy=http://127.0.0.1:10809
sass_binary_site=https://npm.taobao.org/mirrors/node-sass/
registry=https://registry.npm.taobao.org
phantomjs_cdnurl=http://cnpmjs.org/downloads
electron_mirror=https://npm.taobao.org/mirrors/electron/
sqlite3_binary_host_mirror=https://foxgis.oss-cn-shanghai.aliyuncs.com/
profiler_binary_host_mirror=https://npm.taobao.org/mirrors/node-inspector/
chromedriver_cdnurl=https://cdn.npm.taobao.org/dist/chromedriver
```

`.yarnrc`:

``` .yarnrc
registry "https://registry.npm.taobao.org"
sass_binary_site "https://npm.taobao.org/mirrors/node-sass/"
phantomjs_cdnurl "http://cnpmjs.org/downloads"
electron_mirror "https://npm.taobao.org/mirrors/electron/"
sqlite3_binary_host_mirror "https://foxgis.oss-cn-shanghai.aliyuncs.com/"
profiler_binary_host_mirror "https://npm.taobao.org/mirrors/node-inspector/"
chromedriver_cdnurl "https://cdn.npm.taobao.org/dist/chromedriver"
```

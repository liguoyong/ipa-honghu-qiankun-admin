# qiankun-example

qiankun 练习 demo，父应用 vue3(vite)，子应用用 vue(webpack) 和 vue3(webpack)和vue3(vite)

## 一、项目特色
### 1. 开发环境，父级用vite秒级更新，
### 2. 子应用也有为vite的vite加入子应用需使用vite-plugin-qiankun, 注意父级应用不用加
为什么不加?
因为官方qiankun没有兼容vite做为子应用,vite启动方式为module, 而其他的启动为script, 或许后面vite官方兼容了vite做为其子应用后就可以去掉该插件了；

## 二、开始
一键安装所有主子应用的依赖
```
npm i
```

一键启动所有所有应用
```
npm start
```
通过 [http://localhost:8080/](http://localhost:8080/) 访问主应用。

## 三、对于乾坤的一些思考

我是被迫接受qiankun的, 起初是因为开发的一个项目用到了乾坤,  修改bug有时不可能在一个工程中完成, 需要启动N个工程才能看到全貌, 对此我感觉特不方便, 于是我有个想法把乾坤干掉, 我直接加载里面的组件, 它不香么? 要干掉对手, 就得先了解对手, 于是我开始深入了解乾坤, 并对此进行多个尝试....

最终我把乾坤从工程里面干掉了, 采用热门的monorepo方式搭建, 干掉乾坤没花多久时间, 主要改造如下:
1. 所有分包依然保留, 因为分包开发一定是降低工程的理解难度 ,后面维护也方便,  像vue3 打包出来也就1万5千行, 但分了好几个工程,
2. 采用热门的monorepo方式, 之前工程都塞到package里面.
3. 更换统一之前的引用方式, 之前命名方式五花八门, 现在统一 前缀名+ 包名, 比如`@cnbi/chart`(参照vue3的命名方式),
4. 只是把之前的页面, 需要跳转新窗口的, 把它弄成跳转路由, 相关页面路由从其他包的路由里面拿过来, 也就是之前路由分散在各个包中, 我这里就在路由下面新建对应模块, 建立路由.
5. 启动主工程调试
6. 干掉乾坤的相关配置

至此之前需要开多个工程的项目, 整成了一个, 感觉页面都快多了...

其实并不是我认为qiankun不行, 而使用乾坤需要看应用场景,就我个人理解:
 1. 乾坤适合新老项目工程搭配,老工程是老技术实现的, 升级有难度或没必要, 那么这时可以新模块用新技术实现,老工程稍微用乾坤改改
 2. 两个工程技术一样, 但两个工程完全没有什么重合的场景, 但有时需要结合使用,就像我目前这个工程, 两个工程业务场景压根不是你包含我或我包含你的关系, 但又需要同时在一个工程中使用, 整合肯定没必要, 采用乾坤,  分开部署, 分开升级是个不错的选择.
 3. 完全新开工程, 不要试图采用乾坤进行微服务架构, 没有必要, 现在前端工程可以用更先进的monorepo分包模式架构, 分包还可以进行版本升级,打成插件的模式, 进行版本管理, 即使后期应用到不同项目也方便对不同插件进行升级, 整成微服务后, 每个服务都要去维护, 那么维护成本也会很大
## 四、需要注意的坑
我是被这些坑折磨够呛, 这里记录一下,
1.  父应用常规配置vite和乾坤没啥讲的

2. 子应用配vite-plugin-qiankun, main里面也要使用

3. 图片加载失败?? 因为vite的代码没有经过打包,所以不会自动加上域名,在父应用就看到图片挂了!!!百度也查不出个123, 甚至想用axios请求图片吧,看又不是常规vue写法了,不至于吧!!最后仔细又看了一遍配置文档,发现可以配置config.base,指定vite服务域名, 当基座打开浏览时,依然挂的是子应用域名,有时候base还是不生效,这里采用的是图片资源直接放到public里面, 引用图片时加个window.ORIGIN, 这个值是mian里面初始化过的,所有前缀都加上,这样子基座里面请求的图片是子应用带上的

4. 路由跳转出错? 这个也是我粗心,复制别人的 window.__POWERED_BY_QIANKUN__ 然后路由死活不行, 最后看文档要写成qiankunWindow.__POWERED_BY_QIANKUN__

5. 子应用跳转vue3Vite/vue ,什么鬼?路由叠加? 抓狂,原来是unmount里面漏写了history.destroy()

6. axios请求时, 需要主工程也配置相应代理, 否则请求会出现404, 因为开发环境下是主工程做的代理替换

7. 加载子应用的某一个路由? 怎么设定? 在测试过程中, 好几个小时的尝试都是在registerMicroApps 时, 如果name 和activeRule 不匹配时, 就不能进行模板替换,
- name（子应用的名称） 这个好理解, 比如我的子应用叫dash(随意), 后面dash里面的其他路由都会挂在这个后面

- activeRule（子应用的激活规则） /dash/index, 路由命中后,跳转/dash/index路由, 想法很不错, 但是事与愿违, 不出意外跳进去没有问题, 但是当点击该子应用其他按钮跳转路由时就有问题了,跳完页面就被卸载了......为什么会这样子呢? 因为路由匹配机制, /dash/index  和 /dash/view 它会认为不匹配, 直接就卸载了微应用, 但如果我就命名为 /dash   那么/dash/index和/dash/view 它会认为是同一个微应用
- props（主应用需要传递给子应用的数据）

8. 加载子应用失败? 是的, 怎么都不加载.....页面空白, 没有dom:
- 只能一一检查配置, 但确实没有配错
- 其他路由有影响? 干掉所有路由, 加载一个测试的页面..没用
最后检查到public的cdn引用加载响应qiankun的替换, 具体原因没查, 就是下面几个影响, 注释掉之后马上就加载了服务, 高德地图影响qiankun?......
```js
    <script type="text/javascript"
      src="https://webapi.amap.com/maps?v=1.4.15&key=72601eb4f91ef7b47a9b31163e10e37f"></script>
    <script src="https://webapi.amap.com/loca?key=72601eb4f91ef7b47a9b31163e10e37f&v=1.3.2">
    </script>
    <script src="//webapi.amap.com/ui/1.1/main.js?v=1.1.1"></script>
```
9. 我只是加载一个子应用,而且首屏就要加载, 这时候刷新页面存在乾坤找不到dom元素? 因为刷新页面后存在可能路由页面还没加载然后就注册了qiankun子应用,导致找不到替换的id页面,后面查原因是子程序出错了, token过期...之前是写死在里面的这下又要解决单点登陆问题了
还有其他问题, 就如例子中,我把`<div id="cnbi-viewport"></div>`写在about页面, 他属于路由的子页面,并不在App页面中,切换路由后依然报找不到`cnbi-viewport` 这个div,  我又看了下qiankun说明, 可以手动加载`loadMicroApp`, 也出错... 看来只能在 <router-view></router-view> 上面直接用`<div id="cnbi-viewport"></div>`了


10. 单点登陆: 这个可大可小, 大的集团公司, 一般会有一个认证中心, 认证中心会有应用注册--注册完了有个id----授权---路由重定向,等过程. 咱没有认证中心, 那就简单调接口实现?
- A应用登陆后,控制B应用路由页面是否显示, 这算是一道权限控制, 路由页面有了后, 内部调用B应用的登陆(前提两应用用户认证的是一样的)
- token都由A应用传到B应用, 简单暴力, 如果需要运维单独登陆到B系统,那么B应用搞个自己的认证体系,登陆(我采用第二种)

至此改造算是完成了qiankun+vite的基础配置


## 五、父子组件传值
之前demo的传值,是主工程下发到main里面的, 压根不能在vue组件里面使用, 我也看了别人一些写法, 大多是有在子工程里面, 又建一个方法, 这样子其实是冗余的

 主子工程的通信格式肯定是要一致的, 否则就是鸡同鸭讲话, 既然一样那么方法直接定义到主工程里面, 格式也由主工程定义就好了, 派发给子工程用

  子工程在main里面接收, 然后作为方法,直接挂在到全局,`Vue.prototype.$qiankun=props`,  子应用的vue页面通过` this.$qiankun`可以获得所有方法, 然后进行操作, 具体看vue子应用的App页面

<!-- // "start:vue3": "cd applications/OperationalModule && npm start", -->

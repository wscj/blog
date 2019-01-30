npm安装失败，很多情况是因为某些包下载失败导致。npm官方的镜像源在国内访问很慢，安装成功概率很低，所以切换镜像源能极大提高安装的成功率和速度。切换方式很简单，编辑文件`~/.npmrc`（没有该文件就创建），直接指定`registry`就可以了，具体怎么写见下面示例

#### 淘宝源

换成淘宝源能快速安装绝大多数的包，在`~/.npmrc`写入

```bash
registry=https://registry.npm.taobao.org
```

#### 自定义源

自己可以创建npm服务器，然后在项目安装自己私有的包，这时只要把源指向自己的服务即可，在`~/.npmrc`写入

```bash
registry=你的域名或ip端口
```

#### 指定临时源

一般情况下，一个项目除了安装自己的私有包还有其他包，这时可以分开指定临时源来安装，如先安装私有包，在安装剩下的

```bash
$ npm i my-package1 --registry http://npm.my-domain.com
$ npm i my-package2 --registry http://npm.my-domain.com
$ npm i --registry https://registry.npm.taobao.org
```

#### 个别外链下载依赖

有些npm包在安装的时候，需要去从其他地方下载某些文件，下面我整理了我所遇到过的难以下载的文件，并将其源写入`~/.npmrc`文件

```bash
chromedriver_cdnurl=http://cdn.npm.taobao.org/dist/chromedriver
sass_binary_site=https://npm.taobao.org/mirrors/node-sass/
registry=https://registry.npm.taobao.org
ELECTRON_MIRROR=https://npm.taobao.org/mirrors/electron/
```

#### 特殊外链

##### Sharp

sharp是个处理图片很好用得node包，但是安装的时候有个文件很难下载成功，所以官方提供指定外链的方法，可以指定到任意地址。常用的方法就是先在[这里](https://github.com/lovell/sharp-libvips/releases)下载文件，然后放在一个本地服务，将外链指向这个服务就行了，在`~/.npmrc`写入

```bash
SHARP_DIST_BASE_URL=http://localhost:8088/
````

#### 指定临时外链

可以用以下命令指定临时外链安装包，不过一般写入`~/.npmrc`更简单点，不用每次都指定。也有些奇葩情况，比如我一同事的mac，写入`~/.npmrc`不生效，临时指定就生效O(∩_∩)O哈哈~

```bash
$ ELECTRON_MIRROR=https://npm.taobao.org/mirrors/electron/ npm i 
$ ELECTRON_MIRROR=https://npm.taobao.org/mirrors/electron/ npm run build
```

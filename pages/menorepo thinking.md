- menorepo 只要是指多个不同的应用或包放在同一个 codebase 里管理。与之对应的是 polyrepo，多个 codebase 分开处理发布与版本。
-
- menorepo 能解决什么问题呢，主要有以下几点：
- 1. 首先是代码共享，在 ployrepo 中对 sharing code 的处理是十分繁琐的，一般会对这些代码独立做一个仓库生成 npm 包。假设现在应用 A 与应用 B 都依赖了一个公共 npm 包 shared-utils 。假设 shared-utils 存在一个 bug 导致了 A 与 B 都出现了问题。在 polyrepo 里流程可能是这样的，首先需要在 shared-utils 里提代 commit 修复，然后执行 publish 脚本去在 npm registry 里生成一个版本。然后在 A 与 B 中分别升级，提交修复 commit，最后对 A 与 B 执行构建部署。而如果他们都在一个 repo 里，只需要一次修复 commit 然后构建提交即可。本地开发会更为复杂，一般这种情况会把 shared-utils link 到 A 与 B，不能热更新，每次修改代码需要手动重新 link。每次在测试环境部署，都需要上述在 shared-utils 里 commit 并生成版本，然后在应用里再 commit 并部署的过程。如果是 menorepo，本地可以热更新，并且测试环境发布也会更快捷。
  2. 一些基础代码规范配置的一致共享，比如 tsconfig，eslint 等。
-
- menorepo 本身需要依赖 workspace。workspace 是一个由包管理工具提供的概念，npm，yarn，pnpm 都支持 workspace。每个 menorepo 里的 app 或者 pkg 都是一个 workspace，只要在 menorepo 的 root 上声明了 worksapces，多个 workspace 是可以互相引用的。这具体是指互相依赖的 workspace 在 node_modules 中会生成对方目录的软链。比如上面的 shared-utils，A 与 B 对它的引用，都是从它自身的真实目录里去解析的。
-
- 在 menorepo 里的 package 有两种类型，internal packages 与 external packages。internal packages 是指只在 menorepo 内部使用的包，external packages 指会发布到 npm 上的包，就是我们一般意义上理解的包。external packages 通常会有以下的流程
	- **Bundlers** : 打包构建，webpack，rollup ...
	- **Versioning** : 更新 npm 里的 version
	- **Publishing** : 推到 npm registry
- internal packages 由于只在 menorepo 里的其他 worksapce 里用，可以跳过这些步骤，直接源码级引用并参与对应应用的构建即可。唯一需要注意的是正确在 package.json 设置 internal packages 的入口及 ts 定义。由于参与了对应应用的构建，如果对应应用配置了一些热更新方案，比如 `react-refresh`，也可以直接享受到热更新效果。
- 对于 external packages，通常还需要一个 dev 脚本，用于监听代码变更并重新编译，以达到热更新的效果。并且在 package.json 需要设置编译的后结果作为入口。
-
- menorepo 里版本生成相对复杂一些，通常不被包管理工具原生支持。
- 1. [changeset](https://github.com/changesets/changesets) 是个不错的选择。
  2. 老牌的 [lerna](https://github.com/lerna/lerna) 也可以，但是只需要用到 version 和 publish 两个命令，整体依赖太过重量。社区也存在 fork 版本 [lerna-lite](https://github.com/lerna-lite/lerna-lite)。
  3. [bumpp](https://www.npmjs.com/package/bumpp) 比较轻量的 version 方案。
-
- LATER how menorepo versioning works？
-
- 由于有一些依赖会在 menorepo 的 root 下面，容器化部署分别安装 root 和对应 workspace 下的依赖，由于 root 下的依赖可能被会频繁改动，导致 [docker 构建的缓存](https://bitjudo.com/blog/2014/03/13/building-efficient-dockerfiles-node-dot-js/) 难以被复用。一般而言需要 menorepo 方案提供剪枝方法。
-
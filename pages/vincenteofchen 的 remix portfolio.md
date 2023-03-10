- 如何在 remix.run 里集成 markdown content
	- 基于 kent 的一个 remix 模板：
		- https://github.com/Girish21/speed-metal-stack
	- 详细说明各种方案的一篇文章：
		- https://andre-landgraf.dev/blog/2022-05-29_how-to-integrate-markdown-content-with-syntax-highlighting-in-remix/
	- 一个比较简单的博客系统的 youtube 教程
		- https://www.youtube.com/watch?v=5O6uhyaSvCM
-
- 网上大多数 CMS 的方案，要么把 markdown 存数据库，要么存 github，然后代码再去读 github 内容。为啥不直接读文件系统？
	- 因为大多数 serverless 环境，比如 Vercel 与 Netlify 是不支持读取文件系统的。并且这种情况下更新 blog 的时候需要重新部署。
	- 可以直接用 github 当 CMS 用，github 本身提供各种 rest api，可以使用 [octokit](https://octokit.github.io/rest.js/v19/)，需要注意的是一些 api 需要配置 github token
- 为 react-markdown 实现 codeblock 高亮
	- https://amirardalan.com/blog/syntax-highlight-code-in-markdown
- 关于 markdown 样式渲染
	- https://tailwindcss.com/docs/typography-plugin
	- 这个非常 nice
- kent 关于他的技术栈的一些说明
	- https://kentcdodds.com/blog/how-i-built-a-modern-website-in-2021
	- very nice
- 因为访问 github api 速度很慢，所以需要实现 cache 机制：
	- kent 从 redis 迁移到了 litefs，但 fly.io 的 litefs 还处于 beta 阶段
	- https://kentcdodds.com/blog/i-migrated-from-a-postgres-cluster-to-distributed-sqlite-with-litefs
	- https://github.com/Xiphe/cachified
	-
## python自动生成文档
	
	1. 最好的文档就是代码本身，写代码的时候，加上注释，然后使用脚本自动构建文档。
	2. python中的自动文档构建工具
		1. sphinx: 使用reStructredText作为标记语言，类似Markdown
		2. pdoc是一个简单易用的命令行工具，可以生成python中API文档。
		3. Doxygen：老牌文档生成工具。

# Sphinx的使用
	1. 最初是为python文档构建的
	2. 安装：
		pip install -U sphinx
	3. 脚本工具：
		sphinx-quickstart: 快递生成文档，他将生成一个文档的源目录并且使用一些默认值填充他
	4. 配置文件：
		1. 可以在其中配置sphinx如何阅读源代码和构建文档的各个方面。
		2.  
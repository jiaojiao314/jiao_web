# 002 mkdocs 搭建文档博客

# 1 安装
## 1.1 docker安装
```shell
apt  install docker.io

docker run --rm -it -v ${PWD}:/docs squidfunk/mkdocs-material new .
这将创建以下结构：
.
├─ docs/
│  └─ index.md
└─ mkdocs.yml

docker run --rm -it -p 80:8000 -v ${PWD}:/docs squidfunk/mkdocs-material
```


## 1.2 pip安装
```shell
pip install mkdocs-material

mkdocs new .
这将创建以下结构：
.
├─ docs/
│  └─ index.md
└─ mkdocs.yml

Commands
mkdocs new [dir-name] - Create a new project.
mkdocs serve - Start the live-reloading docs server.
mkdocs build - Build the documentation site.
mkdocs -h - Print help message and exit.
Project layout

mkdocs.yml    # The configuration file.
docs/
    index.md  # The documentation homepage.
    ...       # Other markdown pages, images and other files.
    
```

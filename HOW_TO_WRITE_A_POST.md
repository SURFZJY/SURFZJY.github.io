# 如何在 GitHub 网页上新增一篇博客

站点源码在 `main` 分支。写完后提交到 `main`，GitHub Actions 会自动构建并发布到 `gh-pages`。不要直接改 `gh-pages` 分支。

文件名例子：`2026-11-04-superagent-harness-notes.md`，对应地址 `/blog/2026/superagent-harness-notes/`。

## 新建文件

1. 打开仓库 [SURFZJY/SURFZJY.github.io](https://github.com/SURFZJY/SURFZJY.github.io)。
2. 确认左上角分支是 `main`。
3. 进入 `_posts` 目录，点 **Add file** → **Create new file**。
4. 文件名必须是：

```text
YYYY-MM-DD-短标题.md
```

例如 `2026-11-04-hello-world.md`。

- `YYYY-MM-DD` 是日期，要和正文 front matter 里的 `date` 一致。
- 短标题只用小写英文字母、数字和连字符。
- 发布后的地址是 `https://surfzjy.github.io/blog/年/短标题/`。上面这个例子是 [https://surfzjy.github.io/blog/2026/hello-world/](https://surfzjy.github.io/blog/2026/hello-world/)。

## Front matter

文件最上方用 `---` 包住这些字段，然后再写正文：

```yaml
---
layout: post
title: 文章标题
date: 2026-11-04
description: 一两句话，显示在博客列表里
tags:
  - notes
categories:
  - research
---
```

| 字段          | 作用                                              |
| ------------- | ------------------------------------------------- |
| `layout`      | 固定写 `post`。                                   |
| `title`       | 文章标题。                                        |
| `date`        | `YYYY-MM-DD`。列表和文章页都用它排序、显示日期。  |
| `description` | 列表里标题下面的简介。                            |
| `tags`        | 细一些的标签。文章页会链到 `/blog/tag/标签/`。    |
| `categories`  | 更粗的分类。文章页会链到 `/blog/category/分类/`。 |

`tags` 和 `categories` 可以写多个，每项前面一个 `-`。想让某个标签出现在博客首页顶部，再把它加进 `_config.yml` 的 `display_tags` 或 `display_categories`。不加也可以，文章页和归档页仍然有链接。

代码用三个反引号包起来，并写上语言，例如 `python`。行内公式用一对 `$$`，例如 `$$ E = mc^2 $$`。

## 图片

1. 在 `main` 分支进入 `assets/img/`。
2. 点 **Add file** → **Upload files**，上传图片，提交到 `main`。
3. 在文章里这样引用（把文件名换成你的）：

```markdown
![图片说明]({{ "/assets/img/your-image.jpg" | relative_url }})
```

图片放在 `assets/img/`，不要放进 `_posts`。

---
title: Welcome to the NHUG Blog!
description: Announcing the launch of the NHUG Blog
draft: false
date: 2024-01-08
categories:
  - Announcements
  - Blog
authors:
  - negin513
---

# Welcome to the NCAR HPC User Group (NHUG) Blog!

Today, we are excited to announce the launch of the NCAR HPC User Group (NHUG) Blog. This space is dedicated to sharing insights, tips, and tricks about NCAR HPC resources and practices. 

This blog post explains how to contribute a blog article to the NHUG Blog.

<!-- more -->

## Contributing to the NHUG Blog

This blog post explains how to contribute a blog article to the NHUG Blog:

1. Fork the NHUG repository: If you haven't already, fork the NHUG GitHub repository. This step is necessary for first-time contributors.

2. Clone the forked repository and create a new branch from the `main` branch.

3. Create a file under `/docs/blog/posts/` with the following naming convention: `YYYY-MM-DD-<your-post-title>.md`. The filename you choose does'nt affect the URL or title of your post. The URL of your post will be generated based on the `title` field in the front matter of your post.

```
.
├─ docs/
│  └─ blog/
│     ├─ posts/
│     │  └─ hello-world.md
|     |  └─ YYYY-MM-DD-your-post-title.md
│     ├─ .authors.yml
│     └─ index.md
└─ mkdocs.yml
```
4. Start your blog post with a YAML front matter section. The front matter section must include the following fields:

    ```yaml
    ---
    title: <your-post-title>
    description: <your-post-description>
    draft: false
    date: YYYY-MM-DD
    categories:
      - <category-1>
      - <category-2>

    ---
    ```

    The `title` field is the title of your post. The `description` field is a short description of your post. The `draft` field indicates whether your post is a draft or not. If you set `draft` to `true`, your post will not be published. The `date` field is the date of your post. The `categories` field is a list of categories that your post belongs to. [This page](https://squidfunk.github.io/mkdocs-material/plugins/blog/#metadata) provide a list of other metdata available for each post. 

    Prepare the rest of your blog post in Markdown format. You can use the [Markdown Cheatsheet](https://www.markdownguide.org/cheat-sheet/) as a reference.


5. Once you are happy with your blog post, commit your changes and push your branch to your forked repository. Then, open a pull request to merge your branch into the `main` branch of the NHUG repository.

6. Once your pull request is merged, your blog post will be published on the NHUG Blog. Time to celeberate! :tada:
 

## Additional Tips

#### Adding an Excerpt
To create a concise preview of your post in listings, insert `<!-- more -->` where you want the excerpt to end:

```
Intro paragraph(s) of your post.

<!-- more -->

Continuation of your post...
```

#### Adding an Author

To add an author to your post, add the following field to the front matter section of your post:

```yaml
authors:
  - <author-1-identifier>
  - <author-2-identifier>
```

For first-time contributors, add your details to `.authors.yml` under `/docs/blog/` using a unique identifier (like your GitHub username).
Once you have added your name to the `.authors.yml` file, you can add your identifier in the `authors` field in the front matter section of your post.


#### Blog Post Preview

To preview your blog post locally, first install the following dependencies:
```bash
pip install -r requirements.txt
```
Then,run the following command from the root of the NHUG repository to preview your blog post:
```bash
mkdocs serve
```


---

Stay tuned for more posts, and once again, welcome to the NHUG Blog!


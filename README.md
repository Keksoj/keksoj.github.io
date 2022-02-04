# My blog, powered by Hugo

Check out [Hugo](https://gohugo.io), a great blogging tool for coders.

I use the [LoveIt theme](https://github.com/dillonzq/LoveIt)
made by [dillonzq](https://github.com/dillonzq),
a Shanghai-based developper who wrote a neat `config.toml`
especially made for chinese language enthousiasts:

```toml
# Page config
# 文章页面配置
[params.page]
  # whether to hide a page from home page
  # 是否在主页隐藏一篇文章
  hiddenFromHomePage = false
```

### Custom CSS

Can I input this custom CSS somewhere ?

```css
.archive-item-date {
    /* changing witch from 4em to 6em */
    width: 6em;
    text-align: right;
    color: #a9a9b3;
}
```

I tried putting this custom css into `/assets/css/custom.css` but it failed.
So I changed the LoveIt theme directly.
This is bad. Don't do this at home, kids.

### To clone the repo

There is a submodule attached, so be sure to try:

    git clone --recurse-submodules git@github.com:Keksoj/Keksoj.github.io.git

Check that hugo is installed on the machine.
It is packaged for Arch at least.
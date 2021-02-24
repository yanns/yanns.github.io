# http://yanns.github.io/

Source for http://yanns.github.io/

Created with [zola](https://www.getzola.org/)

## preview

```
zola serve
```

## publishing

- push the changes in the `blog_source` branch.
- the changes are [automatically deployed to the `master` branch](https://github.com/yanns/yanns.github.io/actions).

# How this blog was setup

## install zola:

https://www.getzola.org/documentation/getting-started/installation/

## create blog

zola init blog

## theme

```
git submodule add https://github.com/U9H/feather themes/feather
````

`theme = "feather"` in `config.toml`

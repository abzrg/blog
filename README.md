# My website

## Development on a new machine

```sh
git clone --recursive git@github.com:abzrg/blog
cd abzrg/public && git checkout main
```

## To add a new post

```sh
hugo new posts/<asdf>.md
```


## To generate website

```sh
hugo -t PaperMod
```


## Tips

use the following instead of `hugo server`

```sh
alias hss='hugo server --noHTTPCache --disableFastRender'
```

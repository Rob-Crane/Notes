# Vim Notes

## Tabs
* `vim -p` opens files in separate tabs
* From within Vim:
```
:tabe(dit) {file}    edit file in new tab
:tabf(ind) {file}    edit file in new tab, search path for it
:tabc(lose)          close current tab
:tabc(lose) {i}      close i-th tab
:tabonly
:tabs                list tabs
:tabm                move current tab to last tab
:tabm {i}            move current to i-th tab
:tabfirst            go to first tab
:tablast             to to last tab
```
* `:tab` modifer causes commands that would normally split a window to instead open a new tab
* In normal mode, `gt` goes to next tab, `gT` goes to previous, and `{i}gt` goes to i-th tab

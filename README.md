# git rev-list SLOP demo

This repository is a demo of a case where git rev-list outputs a wrong result, because some commits have parents that have a more recent commit date.

The commit graph looks like this:

```plaintext
*   de8dc9e commit-20 on Jan 20, 2020
|\
| * 9278f7e commit-9 on Jan 09, 2020
| * bd5df04 commit-8 on Jan 08, 2020
| * 6c94181 commit-7 on Jan 07, 2020
| * 51f0e61 commit-6 on Jan 06, 2020
| * 6c8c555 commit-5 on Jan 05, 2020
| * 80746d4 commit-4 on Jan 04, 2020
| * 47c5732 commit-3 on Jan 03, 2020
| * c843393 commit-2 on Jan 02, 2020
| * 7e7d375 commit-1 on Jan 01, 2020
* | 9f1eb5a commit-15 on Jan 15, 2020
|/
* 9655a73 commit-14 on Jan 14, 2020
* 7ec4af9 commit-13 on Jan 13, 2020
* 78616fe commit-12 on Jan 12, 2020
* 33f0557 commit-11 on Jan 11, 2020
* 249802d commit-10 on Jan 10, 2020
```

`git rev-list 9278f7e..de8dc9e` is supposed to show all ancestors of `de8dc9e` that are NOT ancestors of `9278f7e` - i.e. we can expect it to output
```
de8dc9e803100f59163e7718ea4ff94037d435bb
9f1eb5aa160dfbb7e30a9a9b418ae6db21447601
```

But the actual output is:

```sh
$ git --version
git version 2.44.0

$ git rev-list 9278f7e..de8dc9e
de8dc9e803100f59163e7718ea4ff94037d435bb
9f1eb5aa160dfbb7e30a9a9b418ae6db21447601
9655a734ba29b41eff5bbbe083219a5802005b11
7ec4af9c0ab166d02bc0bcecdf98f04e3bae6217
78616fe5ba579cec467c865b6bd1708951b55e4a
33f055753686800bad5cd6287a8a5b1ce9e268fb
249802dd9cfe7fc8cf9cf9a2eb134f91555ce011
```

This is because git rev-list will walk commits by commit date (older first).
When the rev-list command encounters 5 (defined [here](https://github.com/git/git/blob/3bd955d26919e149552f34aacf8a4e6368c26cec/revision.c#L1298)) consecutive uninteresting commits, it will stop walking that branch.
In this particular case, git rev-list will walk through `9655a73` and its ancestors first, and it will never be marked as uninteresting because there are more than 5 **older** commits before it can be reached.

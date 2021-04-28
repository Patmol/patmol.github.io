---
title: "Easily validate a checksum on MacOS"
date: 2021-04-28 14:00:00 +0200
categories: [Vue.js](Termincal)
tags: [programming, tutorial, vuejs, unit test, axios](#)
---
> A digit representing the sum of the correct digits in a piece of stored or transmitted digital data, against which later comparisons can be made to detect errors in the data.  [lexico](#)

When you download a file, you usually encounter a checksum value to help you check that the file you've downloaded has not been altered, but I'm thinking that checking this value is never straightforward.
You need to remember which command to use with which parameters and then manually validate that the command's output matches the value provided by the website.

To avoid performing all those steps manually, I've created a small function in my shell.

## How to validate a checksum?
To get the checksum of a file (here the SHA checksum), we need to use the following command `shasum` with the `-a' parameter which will indicate the algorithm used, then the path to the file.
`Example : 
`shasum -a 512 ~/Downloads/dotnet-sdk-5.0.202-osx-x64.pkg`
So, in our case, we will use the SHA512 algorithm on a .pkg file in the Downloads folder.
Once execute, you will have an output like this
```bash
f112dde7a02301a61c05db60800cf3d08989e7d47341d2b63d4d11510049e6bf4c6eebcbf70c767740618e7df0b51d831c8f327c55fb672e28e536de19d569a0  /Users/cedric/Downloads/dotnet-sdk-5.0.202-osx-x64.pkg
```
There is two parts in the output, the first part is the checksum itself and then the path to the file.
To validate the checksum, you need to manually compare the first part with the value provided by the website.

## Create a function to perform the validation
Relatively easy but still dull to do manually. 
Why not just have a function with another parameter to put the checksum we must have and tell if it's the same or not?
That's what we are going to do :-)

Let's create a file name `checksum` . Remind yourself where you create this file because you will need it later (I've created this file under the path `~/.zshfn/`). 

Once it's done, open it with your favorite text editor and add this content

```bash
#!/bin/bash
function checksum() {
    diff <(shasum -a "$1" "$2" | cut -d ' ' -f 1) <(echo "$3")
}
```

What is this function doing?
- `shasum -a "$1" "$2"` We have the checksum function (just like we did in the last part but where a function parameter replaces each parameter.
- `| cut -d'' -f 1` take the output of the shamus function and cut it on the space character, then take the first element (so, the checksum itself)
- `diff <() <(echo "$3")` check the difference between the first string and a second one send as the third parameter of the checksum function we are creating.

Now that we have written our custom function, we can load it in our shell with this command `source ~/.zshfn/checksum` (you will need to replace `~/.zshfn/` with the path under which you've created your file previously).
And then, you can test the function by executing this command
```bash
checksum 512 ~/Downloads/dotnet-sdk-5.0.202-osx-x64.pkg f112dde7a02301a61c05db60800cf3d08989e7d47341d2b63d4d11510049e6bf4c6eebcbf70c767740618e7df0b51d831c8f327c55fb672e28e536de19d569a0
```
If the checksum is valid, nothing will be displayed but if you have a difference between the checksum of the file and the one you need to match, you will have something like this (both checksum)
```bash
< f112dde7a02301a61c05db60800cf3d08989e7d47341d2b63d4d11510049e6bf4c6eebcbf70c767740618e7df0b51d831c8f327c55fb672e28e536de19d569a0
---
> f112dde7a02301a61c05db60800cf3d08989e7d47341d2b63d4d11510049e6bf4c6eebcbf70c767740618e7df0b51d831c8f327c55fb672e28e536de19d569ae
```
Thanks to the checksum function, you can easily see if the file is valid or not. 

## Automatically load the function
_Those steps will depend on the shell you are using._
There is one little step to perform, because if you close and relaunch your shell, the checksum function will not be loaded, so we will need to update a file to automatically load it.
If you are using Zsh, here are the steps you need to perform.
Firstly, you can create (or edit if this file is already existing) the following file `~/.zshrc`.
And at the end of the file, you need to add this content (change the path if you've used something different than `~/zshfn`)
```bash
fpath=( ~/.zshfn "${fpath[@]}" )
autoload -Uz checksum
```

If you want to automatically load all functions in the `~/.zshfn` path, you can change the last line by this `autoload -Uz $fpath[1]/*(.:t)`

Save the file and restart your shell.

The `checksum` function is now available, and you can start to validate the checksum of each file you download, so you can be sure to have the correct file and not a corrupt one :-) 
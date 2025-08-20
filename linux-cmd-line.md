## Permissions in Linux 
3 groups (owner, groups, others)
'-' means file
'd' means directory

You can change permissions if you own the file or you are root.
There is 10 string character that defines
the permissions of a file/directory:
`-rwxrwxrwx` this means all the groups (owner, groups, others) have (r)read,
(w)write,(x)execute permissions for the file.
`drwxr-xr-x`  this means in the directory owner has read, write, execute
permission. Groups and others have only read and execute permission.

## Change owner
As you know from above you can only change permission if you own the 
file/directory or you are root

You can also change the owner as follows:

```bash
# changing the owner to root for directory 'contacts'
sudo chown -R root contacts
```

## `xargs`
Sometimes there are some tools that don't work with pipe operator `|`

`xargs` takes input from a pipe and converts it into arguments for
another command. It's especially useful when you have a command that
doesn't directly accept input from pipes.

```bash
# this will not work
echo "file.txt" | cat

# this will work
echo "file.txt" | xargs cat
```

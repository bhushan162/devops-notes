# How we can build project 

```cmd
west build -p always -b <your-board-name> samples/basic/blinky

# -p always: prestine build, its like remove build folder before build. 
# -p auto : west will decide clean build or no
```


## flash the sample 

```
west flash 
```

## VERSION file
- this is file at root lavel of project that we stiore version in following Format
```sh
    VERSION_MAJOR =
    VERSION_MINOR =
    PATCHLEVEL =
    VERSION_TWEAK =
    EXTRAVERSION =
```

e.g. 
```sh
    VERSION_MAJOR = 1
    VERSION_MINOR = 2
    PATCHLEVEL = 3
    VERSION_TWEAK = 4
    EXTRAVERSION = unstable.5
```

##

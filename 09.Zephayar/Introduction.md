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

-e

-i

s

s/xxx/yyy/g

s;xxx;yyy;

`find . -name "subdir.mk" -exec sed -i 's;$(CC) $(BUILD_CFLAGS);$(CC) $(BUILD_CFLAGS) -fPIC;' '{}' \;`

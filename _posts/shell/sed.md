-e，不写入文件

-i，写入文件

s，替换

s/xxx/yyy/g

s;xxx;yyy;

所有 subdir.mk 替换 "$(CC) $(BUILD_CFLAGS)" 为 “$(CC) $(BUILD_CFLAGS) -fPIC”，`find . -name "subdir.mk" -exec sed -i 's;$(CC) $(BUILD_CFLAGS);$(CC) $(BUILD_CFLAGS) -fPIC;' '{}' \;`

所有 .c 文件首行添加 “#include <asm-generic/io.h>”，`find . -name "*.c" -exec sed -i '1 i#include <asm-generic/io.h>' "{}" \;`

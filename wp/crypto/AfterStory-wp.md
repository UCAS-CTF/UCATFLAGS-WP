### 题目源码

`AfterStory.c`

{% raw %}
```c
# include "AfterStory.h"
# define checkfile(f, msg, op) if(!f){perror(msg);op;return 1;}

int main() {
    AfterStory myStory;

    FILE *f = fopen("Flag", "rb");
    checkfile(f, "Error opening file", );

    FILE *c = fopen("Cipher", "wb");
    checkfile(c, "Error opening file", fclose(f));

    start_Story(&myStory, INIT, 0x8888888888888888);

    unsigned char buffer[1024];
    ull rand_num;
    size_t bytes_read;

    while ((bytes_read = fread(buffer, 1, sizeof(buffer), f)) > 0) {
        for (size_t i = 0; i < bytes_read; ++i) {
            if (sizeof(rand_num) == 8) {
                rand_num = myStory.init;
                buffer[i] ^= rand_num & 0xFF;
                for (size_t j = 1; j < sizeof(rand_num); ++j) next_Story(&myStory);
            } else {
                break;
            }
        }
        fwrite(buffer, 1, bytes_read, c);
    }

    fclose(f);
    fclose(c);

    return 0;
}
```
{% endraw %}


`AfterStory.h`

{% raw %}
```c
# ifndef LFSR_H
# define LFSR_H

# include <stdio.h>
# include "Secret.h"
# define ull unsigned long long

typedef struct lfsr {
    ull init;
    ull mask;
} AfterStory;

void start_Story(AfterStory *lfsr, ull init_seed, ull polynomial) {
    lfsr -> init = init_seed;
    lfsr -> mask = polynomial;
}

void next_Story(AfterStory *lfsr) {
    ull bit = 0;
    ull num = lfsr -> init & lfsr -> mask;
    while (num) {
        bit ^= 1;
        num &= num - 1;
    }
    lfsr -> init = (lfsr -> init << 1) | bit;
}

# endif
```
{% endraw %}

### 思路分析

注意到 Flag 的格式一定以 `ucatflags{` 开头，而密钥（lfsr产生的数据）是直接异或到 Flag 上的，所以可以获得密钥的前几十位。这几十位恰好能覆盖到初始状态的位数，因此可以直接利用当前获得的前几十位往后使用给定的lfsr进行递推。

### 解题代码

`solve5.c`

{% raw %}
```c
# include "solve5.h"
# define checkfile(f, msg, op) if(!f){perror(msg);op;return 1;}

int main() {
    AfterStory myLFSR;

    FILE *f = fopen("Cipher", "rb");
    checkfile(f, "Error opening file", );
    printf("%p\n", f);

    FILE *c = fopen("Flag", "wb");
    checkfile(c, "Error opening file", fclose(f));

    unsigned char key[10];
    unsigned char kbytes[] = "ucatflags{";
    fread(key, 1, 10, f);
    for (int i = 0; i < 10; ++i) printf("%x ", key[i]);
    printf("\n");
    for (int i = 0; i < 10; ++i) key[i] ^= kbytes[i];
    
    ull INIT = key[0];
    for (int i = 1; i < 9; ++i) INIT = (INIT << 7) | (key[i] & 0x7F);

    lfsr_init(&myLFSR, INIT, 0x8888888888888888);

    unsigned char buffer[1024];
    ull rand_num;
    size_t bytes_read;

    // 注意文件指针复位
    fseek(f, 0, SEEK_SET);
    while ((bytes_read = fread(buffer, 1, sizeof(buffer), f)) > 0) {
        for (size_t i = 0; i < bytes_read; ++i) {
            if (sizeof(rand_num) == 8) {
                rand_num = myLFSR.init;
                printf("%x ", (unsigned char)buffer[i]);
                buffer[i] ^= (rand_num >> (64 - 8)) & 0xFF;
                for (size_t j = 1; j < sizeof(rand_num); ++j) lfsr_next(&myLFSR);
            } else {
                break;
            }
        }
        printf("\n");
        fwrite(buffer, 1, bytes_read, c);
    }

    fclose(f);
    fclose(c);

    return 0;
}
```
{% endraw %}

`solve5.h`

{% raw %}
```c
# ifndef LFSR_H
# define LFSR_H

# include <stdio.h>
# define ull unsigned long long

typedef struct lfsr {
    ull init;
    ull mask;
} AfterStory;

void lfsr_init(AfterStory *lfsr, ull init_seed, ull polynomial) {
    lfsr -> init = init_seed;
    lfsr -> mask = polynomial;
}

void lfsr_next(AfterStory *lfsr) {
    ull bit = 0;
    ull num = lfsr -> init & lfsr -> mask;
    while (num) {
        bit ^= 1;
        num &= num - 1;
    }
    lfsr -> init = (lfsr -> init << 1) | bit;
}

# endif
```
{% endraw %}

`Makefile`

{% raw %}
```Makefile
CXX = g++
CXXFLAGS = -Wall -Wextra
LDFLAGS =
TARGET = solve5
SOURCES = solve5.c
OBJECTS = $(SOURCES:.c=.o)
all: $(TARGET)
$(TARGET): $(OBJECTS)
	$(CXX) $(LDFLAGS) $(OBJECTS) -o $(TARGET)
%.o: %.c
	$(CXX) $(CXXFLAGS) -c $< -o $@
clean:
	rm -f $(TARGET) $(OBJECTS)
decrypt:
	./$(TARGET)

.PHONY: all clean
```
{% endraw %}

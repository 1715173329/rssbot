# rssbot

中文 Telegram RSS 机器人 [@RustRssBot](http://t.me/RustRssBot)

## 使用

    /rss   - 显示当前订阅的 RSS 列表，加 raw 参数显示链接
    /sub   - 订阅一个 RSS: /sub http://example.com/feed.xml
    /unsub - 退订一个 RSS: /unsub http://example.com/feed.xml

## 编译

先安装 *Rust Nightly* 以及 *Cargo* (推荐使用 [`rustup`](https://www.rustup.rs/)), 然后:

```
cargo build --release
```

编译好的文件位于: `./target/release/rssbot`

## 运行

```
./rssbot DATAFILE TELEGRAM-BOT-TOKEN
```

`DATAFILE` 为数据库保存路径(其实就是一个 json 文件), `TELEGRAM-BOT-TOKEN` 请参照 [这里](https://core.telegram.org/bots#3-how-do-i-create-a-bot) 申请

## 从旧的 RSSBot 迁移

对于 [原先 Clojure 版本的 Bot](https://github.com/iovxw/tg-rss-bot), 可以使用以下脚本转换数据库

```bash
#!/bin/bash

DATABASE=$1
TARGET=$2

DATA=$(echo "SELECT url, title FROM rss;" | sqlite3 $DATABASE)
IFS=$'\n'

echo -e "[\c" > $TARGET
for line in ${DATA[@]}
do
    IFS='|'
    r=($line)
    link=${r[0]}
    title=${r[1]}

    echo -e "{\"link\":\"$link\"," \
            "\"title\":\"$title\"," \
            "\"error_count\":0," \
            "\"hash_list\":[]," \
            "\"subscribers\":[\c" >> $TARGET

    subscribers=$(echo "SELECT subscriber FROM subscribers WHERE rss='$link';" | sqlite3 $DATABASE)
    IFS=$'\n'
    for subscriber in ${subscribers[@]}
    do
        echo -e "$subscriber,\c" >> $TARGET
    done

    echo -e "]},\c" >> $TARGET
done
echo "]" >> $TARGET
sed -i "s/,]/]/g" $TARGET
```

参数 1 为旧数据库地址, 2 为结果输出地址

需要注意的是已推送的 RSS 记录不会保留, 如果直接使用转换后的数据库, 会重复推送旧的 RSS

## License

This is free and unencumbered software released into the public domain.

Anyone is free to copy, modify, publish, use, compile, sell, or distribute this software, either in source code form or as a compiled binary, for any purpose, commercial or non-commercial, and by any means.

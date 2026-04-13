# SQL

## トランザクションテーブル(プレフィックスがt_で始まるテーブル)のDELETE文を一括発行

```sql
SELECT CONCAT('DELETE FROM ', table_name, ';') AS del_stmt
FROM information_schema.tables
WHERE table_schema = DATABASE()
  AND table_name LIKE 't\_%';
```

## 全テーブルの件数を取得するスクリプト

```sh
#!/bin/sh

# MySQLクライアントのインストールの必要あり
# apt update
# apt install default-mysql-client -y

# MySQL接続情報
USER="myuser"
PASSWORD="password"
HOST="mysql"
DATABASE="db_pxc"

# テーブル一覧を取得
tables=$(mysql -u"$USER" -p"$PASSWORD" -h "$HOST" -D "$DATABASE" -Bse "SHOW TABLES;")

echo "Table Name | Row Count"
echo "---------------------"

# 各テーブルの件数を取得
for table in $tables; do
    count=$(mysql -u"$USER" -p"$PASSWORD" -h "$HOST" -D "$DATABASE" -Bse "SELECT COUNT(*) FROM \`$table\`;")
    echo "$table | $count"
done
```

## スクリプト実行毎にテーブルの件数の差分を検知するスクリプト

```sh
#!/bin/bash

# MySQLクライアントのインストールの必要あり
# apt update
# apt install default-mysql-client -y

# MySQL接続情報
USER="myuser"
PASSWORD="password"
HOST="mysql"
PORT="3306"
DATABASE="db_pxc"

# スクリプト自身のディレクトリを取得
SCRIPT_DIR=$(dirname "$(readlink -f "$0")")

# 保存ファイル
PREV_COUNT_FILE="$SCRIPT_DIR/${DATABASE}_table_counts.txt"
CURRENT_COUNT_FILE="$SCRIPT_DIR/${DATABASE}_table_counts_current.txt"
DIFF_RESULT_FILE="$SCRIPT_DIR/${DATABASE}_table_diff.txt"

# テーブル一覧取得
tables=$(mysql -u"$USER" -p"$PASSWORD" -h "$HOST" -P "$PORT" -D "$DATABASE" -Bse "SHOW TABLES;")

# 現在の件数を取得してファイルに保存
> "$CURRENT_COUNT_FILE"
for table in $tables; do
    count=$(mysql -u"$USER" -p"$PASSWORD" -h "$HOST" -P "$PORT" -D "$DATABASE" -Bse "SELECT COUNT(*) FROM \`$table\`;")
    echo "$table $count" >> "$CURRENT_COUNT_FILE"
done

# 差分結果ファイルを初期化
> "$DIFF_RESULT_FILE"

# 前回ファイルが存在する場合のみ差分を比較
if [ -f "$PREV_COUNT_FILE" ]; then
    while read -r line; do
        table=$(echo $line | awk '{print $1}')
        current_count=$(echo $line | awk '{print $2}')
        prev_count=$(grep "^$table " "$PREV_COUNT_FILE" | awk '{print $2}')
        if [ "$current_count" -gt "$prev_count" ]; then
            diff=$((current_count - prev_count))
            echo "$table increased by $diff (from $prev_count to $current_count)" >> "$DIFF_RESULT_FILE"
        fi
    done < "$CURRENT_COUNT_FILE"

    # 結果があれば標準出力にも表示
    if [ -s "$DIFF_RESULT_FILE" ]; then
        cat "$DIFF_RESULT_FILE"
    fi
fi

# 今回の件数を次回用に保存
mv "$CURRENT_COUNT_FILE" "$PREV_COUNT_FILE"
```

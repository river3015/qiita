---
title: docker-entrypoint.shについて少し調べた
tags:
  - 初心者
  - Docker
private: false
updated_at: '2024-12-15T17:40:22+09:00'
id: 490a76fe9b910c9d908a
organization_url_name: null
slide: false
ignorePublish: false
---
# きっかけ
以前、MySQLコンテナの構築を行いました。
MySQLのデータベースやユーザー設定は、環境変数を定義するだけで出来ました。
* dockerコマンドの引数
* docker-compose.yaml
* kubernetesのマニフェスト内(secret内)

KubernetesでWordPressを構築してみた際に、データベースやユーザーがうまく作成されずハマっていました。
その際に、そもそもどのようにして環境変数として渡すだけで、データベースやユーザーの設定がされるのか調べてみました。

環境変数の例
* MYSQL_ROOT_HOST
*	MYSQL_DATABASE
*	MYSQL_USER
*	MYSQL_PASSWORD
*	MYSQL_ROOT_PASSWORD

結論としては、docker-entrypoint.shで行われていました。

# docker-entrypoint.shとは

docker-entrypoint.sh は、Dockerイメージにおいてコンテナの起動時に実行されるスクリプトです。
このスクリプトを利用することで、コンテナ起動時に必要な初期化処理や設定を行うことができます。
※ChatGPTより作成

# 調べたこと

`mysql:8.0.27`でデータベースやユーザーの設定がどう行われているか調べました。

https://hub.docker.com/layers/library/mysql/8.0.27/images/sha256-238cf050a7270dd6940602e70f1e5a11eeaf4e02035f445b7f613ff5e0641f7d

https://github.com/docker-library/mysql/blob/15fe7357b165ee8aaa3ce165386f910a53a75087/8.0/docker-entrypoint.sh

詳細については不明ですが、実際に`docker-entrypoint.sh`で行われているということは分かりました。

`docker-entrypoint.sh`の248行目`docker_setup_db`より引用
```
# Initializes database with timezone info and root password, plus optional extra db/user
docker_setup_db() {
	# Load timezone info into database
	if [ -z "$MYSQL_INITDB_SKIP_TZINFO" ]; then
		# sed is for https://bugs.mysql.com/bug.php?id=20545
		mysql_tzinfo_to_sql /usr/share/zoneinfo \
			| sed 's/Local time zone must be set--see zic manual page/FCTY/' \
			| docker_process_sql --dont-use-mysql-root-password --database=mysql
			# tell docker_process_sql to not use MYSQL_ROOT_PASSWORD since it is not set yet
	fi
	# Generate random root password
	if [ -n "$MYSQL_RANDOM_ROOT_PASSWORD" ]; then
		export MYSQL_ROOT_PASSWORD="$(pwgen -1 32)"
		mysql_note "GENERATED ROOT PASSWORD: $MYSQL_ROOT_PASSWORD"
	fi
	# Sets root password and creates root users for non-localhost hosts
	local rootCreate=
	# default root to listen for connections from anywhere
	if [ -n "$MYSQL_ROOT_HOST" ] && [ "$MYSQL_ROOT_HOST" != 'localhost' ]; then
		# no, we don't care if read finds a terminating character in this heredoc
		# https://unix.stackexchange.com/questions/265149/why-is-set-o-errexit-breaking-this-read-heredoc-expression/265151#265151
		read -r -d '' rootCreate <<-EOSQL || true
			CREATE USER 'root'@'${MYSQL_ROOT_HOST}' IDENTIFIED BY '${MYSQL_ROOT_PASSWORD}' ;
			GRANT ALL ON *.* TO 'root'@'${MYSQL_ROOT_HOST}' WITH GRANT OPTION ;
		EOSQL
	fi

	local passwordSet=
	if [ "$MYSQL_MAJOR" = '5.6' ]; then
		# no, we don't care if read finds a terminating character in this heredoc (see above)
		read -r -d '' passwordSet <<-EOSQL || true
			DELETE FROM mysql.user WHERE user NOT IN ('mysql.sys', 'mysqlxsys', 'root') OR host NOT IN ('localhost') ;
			SET PASSWORD FOR 'root'@'localhost'=PASSWORD('${MYSQL_ROOT_PASSWORD}') ;

			-- 5.5: https://github.com/mysql/mysql-server/blob/e48d775c6f066add457fa8cfb2ebc4d5ff0c7613/scripts/mysql_secure_installation.sh#L192-L210
			-- 5.6: https://github.com/mysql/mysql-server/blob/06bc670db0c0e45b3ea11409382a5c315961f682/scripts/mysql_secure_installation.sh#L218-L236
			-- 5.7: https://github.com/mysql/mysql-server/blob/913071c0b16cc03e703308250d795bc381627e37/client/mysql_secure_installation.cc#L792-L818
			-- 8.0: https://github.com/mysql/mysql-server/blob/b93c1661d689c8b7decc7563ba15f6ed140a4eb6/client/mysql_secure_installation.cc#L726-L749
			DELETE FROM mysql.db WHERE Db='test' OR Db='test\_%' ;
			-- https://github.com/docker-library/mysql/pull/479#issuecomment-414561272 ("This is only needed for 5.5 and 5.6")
		EOSQL
	else
		# no, we don't care if read finds a terminating character in this heredoc (see above)
		read -r -d '' passwordSet <<-EOSQL || true
			ALTER USER 'root'@'localhost' IDENTIFIED BY '${MYSQL_ROOT_PASSWORD}' ;
		EOSQL
	fi

	# tell docker_process_sql to not use MYSQL_ROOT_PASSWORD since it is just now being set
	docker_process_sql --dont-use-mysql-root-password --database=mysql <<-EOSQL
		-- What's done in this file shouldn't be replicated
		--  or products like mysql-fabric won't work
		SET @@SESSION.SQL_LOG_BIN=0;

		${passwordSet}
		GRANT ALL ON *.* TO 'root'@'localhost' WITH GRANT OPTION ;
		FLUSH PRIVILEGES ;
		${rootCreate}
		DROP DATABASE IF EXISTS test ;
	EOSQL

	# Creates a custom database and user if specified
	if [ -n "$MYSQL_DATABASE" ]; then
		mysql_note "Creating database ${MYSQL_DATABASE}"
		docker_process_sql --database=mysql <<<"CREATE DATABASE IF NOT EXISTS \`$MYSQL_DATABASE\` ;"
	fi

	if [ -n "$MYSQL_USER" ] && [ -n "$MYSQL_PASSWORD" ]; then
		mysql_note "Creating user ${MYSQL_USER}"
		docker_process_sql --database=mysql <<<"CREATE USER '$MYSQL_USER'@'%' IDENTIFIED BY '$MYSQL_PASSWORD' ;"

		if [ -n "$MYSQL_DATABASE" ]; then
			mysql_note "Giving user ${MYSQL_USER} access to schema ${MYSQL_DATABASE}"
			docker_process_sql --database=mysql <<<"GRANT ALL ON \`${MYSQL_DATABASE//_/\\_}\`.* TO '$MYSQL_USER'@'%' ;"
		fi
	fi
}
```

`CREATE USER`や`GRANT`、`ALTER USER`などが書かれているのが分かります。

# おわりに
今後もハマった際にはコンテナ以前の構築手順などと比較して原因を探ってみようと思います。

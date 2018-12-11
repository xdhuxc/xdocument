### DynamoDB 命令行

1、显示所有数据库表
```angular2html
aws dynamodb list-tables --endpoint-url http://52.221.216.74:8000
```

2、显示特定表信息
```angular2html
aws dynamodb describe-table --table-name xdhuxc-test --endpoint-url http://52.221.216.74:8000
```

3、创建表
```angular2html
aws dynamodb create-table \
    --table-name sgt_hawkeye_business \
    --attribute-definitions \
        AttributeName=id,AttributeType=S \
    --key-schema \
        AttributeName=id,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=10 \
    --endpoint-url http://52.221.216.74:8000
```















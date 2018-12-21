### DynamoDB 命令行

1、显示所有数据库表
```angular2html
aws dynamodb list-tables --endpoint-url http://192.168.0.128:8000
```

2、显示特定表信息
```angular2html
aws dynamodb describe-table --table-name xdhuxc-test --endpoint-url http://192.168.0.128:8000
```

3、创建表
```angular2html
aws dynamodb create-table \
    --table-name business \
    --attribute-definitions \
        AttributeName=key,AttributeType=S \
    --key-schema \
        AttributeName=key,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=10 \
    --endpoint-url http://192.168.0.128:8000
```

4、查看 dynamodb 数据库中所有数据
```angular2html
aws dynamodb scan --table-name xdhuxc --endpoint-url http://192.168.0.128:8000
```


```angular2html
aws dynamodb create-table \
    --table-name sgt_hawkeye_business \
    --attribute-definitions \
        AttributeName=key,AttributeType=S \
    --key-schema \
        AttributeName=key,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=10 \
    --endpoint-url http://localhost:8000
```


```angular2html
aws dynamodb put-item --table-name sgt_hawkeye_business --item '{"key":{"S":"/sgt_hawkeye/business_json/1"}, "value":{"S":{"name": "arn:aws:elasticloadbalancing:ap-southeast-1:848318613114:targetgroup/TG-ADS-adx-prod/6daace00d164b398", "labels": {"group": "ADS", "project": "adx", 'env": "prod"}, "port": 10106 }}}' --endpoint-url http://localhost:8000
```

















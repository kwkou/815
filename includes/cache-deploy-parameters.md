
### cacheSKUName

新 Azure Redis 缓存的定价层。

    "cacheSKUName": {
      "type": "string",
      "allowedValues": [
        "Basic",
        "Standard"
      ],
      "defaultValue": "Basic",
      "metadata": {
        "description": "The pricing tier of the new Azure Redis Cache."
      }
    },

模板将定义此参数允许的值（Basic 或 Standard），如果未指定任何值，则分配默认值 (Standard)。Basic 提供单个节点，该节点具有多种大小，最大大小为 53 GB。Standard 提供“主/副本”两个节点，这些节点具有多种大小（最大 53 GB）并提供 99.9% SLA。

### cacheSKUFamily

SKU 的系列。

    "cacheSKUFamily": {
      "type": "string",
      "allowedValues": [
        "C"
      ],
      "defaultValue": "C",
      "metadata": {
        "description": "The family for the sku."
      }
    },

### cacheSKUCapacity

新 Azure Redis 缓存实例的大小。

    "cacheSKUCapacity": {
      "type": "int",
      "allowedValues": [
        0,
        1,
        2,
        3,
        4,
        5,
        6
      ],
      "defaultValue": 0,
      "metadata": {
        "description": "The size of the new Azure Redis Cache instance. "
      }
    }

模板将定义此参数允许的值（0、1、2、3、4、 5 或 6），如果未指定任何值，则分配默认值 (1)。这些数字对应于以下缓存大小：0 = 250 MB，1 = 1 GB，2 = 2.5 GB，3 = 6 GB，4 = 13 GB，5 = 26 GB，6 = 53 GB


<!---HONumber=74-->
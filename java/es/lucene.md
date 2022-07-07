

```json
"query": {
    "bool": {
      "should": [
        {
          "bool": {
            "must": [
              {
                "terms": {
                  "operation_type": ["user/me"]
                }
              },
              {
                "terms": {
                  "user":["candili"]
                }
              },
              {
                "wildcard": {
                  "operation_type": {
                    "value": "*user*"
                  }
                }
              }
            ]
          }
        },
        {
          "bool": {
            "must": [
              {
                "terms": {
                  "operation_type": ["user/menu"]
                }
              },
              {
                "terms": {
                  "user":["tristanzeng"]
                }
              },
              {
                "wildcard": {
                  "operation_type": {
                    "value": "*user*"
                  }
                }
              }
            ]
          }
        }
      ]
    }
  }
```


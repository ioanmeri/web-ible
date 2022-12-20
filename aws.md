## S3 Cross-origin resource sharing (CORS)

1.

```
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "GET",
            "HEAD"
        ],
        "AllowedOrigins": [
            "*"
        ],
        "ExposeHeaders": [],
        "MaxAgeSeconds": 3000
    }
]
```

2.

```
[
  {
    "AllowedMethods": [
      "GET",
      "HEAD"
    ],
    "AllowedOrigins": [
      "https://mistymap.com"
    ],
    "ExposeHeaders": [
      "Access-Control-Allow-Origin"
    ]
  }
]
```

## Check headers with CURL

```
curl -i -H "Origin: https://mistymap.com" https://d2aibnd4ff9k7t.cloudfront.net/styles/monochrome-dark/tiles/256/4/2/2@2x
```

Response:

```
HTTP/2 200
content-type: webp
content-length: 12968
date: Sun, 18 Dec 2022 19:47:23 GMT
access-control-allow-origin: https://mistymap.com
access-control-allow-methods: GET, HEAD
access-control-expose-headers: Access-Control-Allow-Origin
access-control-allow-credentials: true
last-modified: Sun, 11 Dec 2022 14:38:11 GMT
etag: "740ec4db235de455b6258901e3f40aa0"
accept-ranges: bytes
server: AmazonS3
x-cache: Miss from cloudfront
via: 1.1 5d5650d27c767174762251d7b9000c4a.cloudfront.net (CloudFront)
x-amz-cf-pop: ATH50-C1
alt-svc: h3=":443"; ma=86400
x-amz-cf-id: i2Xmh2FJICVPDcBVHdo6ZNB9XJi8sxN5ZTJW03vREXDpxmk3pkfBXQ==
```

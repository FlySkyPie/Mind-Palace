# AWS DNS 域名格式與命名規則

## 概述

AWS 各服務的 DNS 域名遵循特定模式，通常嵌入**區域代碼 (region code)** 與 **AWS 專屬頂級域 (amazonaws.com 或 ip.aws)**。以下依服務類別整理其格式與規則。

## 1. EC2 執行個體

### 公用 DNS（Public DNS）

需 VPC 同時啟用 `enableDnsHostnames` 與 `enableDnsSupport`[^pubdns-req]。

**IPv4 主機名（傳統格式）：**

```
ec2-<public-IPv4-with-dashes>.<region>.compute.amazonaws.com
```

範例：`ec2-52-54-55-66.ap-southeast-2.compute.amazonaws.com`[^pubipv4ex]

- 將 IPv4 位址的 `.` 換成 `-`，前面加上 `ec2-` 前綴
- 使用 split-horizon DNS：VPC 內部解析為私有 IP，外部解析為公用 IP[^splitdns]

**IPv6 主機名（新世代 `ip.aws` 域）：**

```
<base36-ipv6>.<region>.ip.aws
```

範例：`f5lnz-0khrm-nt2u3-gyqqt-nbdl5.ap-southeast-2.ip.aws`[^ipv6ex]

**雙協定主機名：**

```
<base36-ipv6>-<base36-ipv4>.<region>.ip.aws
```

範例：`f5lnz-0khrm-nt2u3-gyqqt-nbdl5-q3cdpO.ap-southeast-2.ip.aws`[^dualex]

> 如果主 IP 變更（stop/start 重新指派），主機名會隨之改變，舊名稱失效[^ipchange]。

---

### 私有 DNS（Private DNS）

**IP 主機名（傳統，基於私有 IPv4）：**

| 區域 | 格式 | 範例 |
|---|---|---|
| `us-east-1` | `ip-<private-ip-with-dashes>.ec2.internal` | `ip-10-24-34-0.ec2.internal`[^useast1int] |
| 其他區域 | `ip-<private-ip-with-dashes>.<region>.compute.internal` | `ip-10-24-34-0.us-west-2.compute.internal`[^othint] |

**資源主機名（較新，基於執行個體 ID，可解析 A 和/或 AAAA 記錄）：**

| 區域 | 格式 | 範例 |
|---|---|---|
| `us-east-1` | `i-<instance-id>.ec2.internal` | `i-0123456789abcdef.ec2.internal`[^rbnuseast] |
| 其他區域 | `i-<instance-id>.<region>.compute.internal` | `i-0123456789abcdef.us-west-2.compute.internal`[^rbnoth] |

- 資源主機名在僅 IPv6 子網中自動啟用[^rbnipv6]
- Amazon DNS 伺服器位址：`169.254.169.253`（IPv4）、`fd00:ec2::253`（IPv6）、VPC CIDR base + 2[^dnsaddr]

---

## 2. RDS 資料庫

**標準資料庫執行個體端點：**

```
<db-instance-identifier>.<12-char-fixed-account-region-id>.<region>.rds.amazonaws.com
```

範例：`mydb.123456789012.us-east-1.rds.amazonaws.com`[^rdsstd]

- `<db-instance-identifier>`：1–63 個字母數字或連字號，首字須為字母，不可以連字號結尾或連續兩個連字號[^rdsnaming]
- `<12-char-fixed-account-region-id>`：同一帳戶 + 同一區域所有執行個體共享此固定 ID，永不變更[^rdsfixid]

**Aurora 叢集端點：**

| 類型 | 格式 | 範例 |
|---|---|---|
| 叢集（寫入器） | `<cluster>.cluster-<random>.<region>.rds.amazonaws.com` | `mydbcluster.cluster-c7tj4example.us-east-1.rds.amazonaws.com`[^auroraw] |
| 讀取器 | `<cluster>.cluster-ro-<random>.<region>.rds.amazonaws.com` | `mydbcluster.cluster-ro-c7tj4example.us-east-1.rds.amazonaws.com`[^aurorar] |
| 執行個體 | `<instance>.<random>.<region>.rds.amazonaws.com` | `mydbinstance.c7tj4example.us-east-1.rds.amazonaws.com`[^aurorai] |
| 自訂 | `<name>.cluster-custom-<random>.<region>.rds.amazonaws.com` | `myendpoint.cluster-custom-c7tj4example.us-east-1.rds.amazonaws.com`[^aurorac] |

**RDS Proxy 端點：** `<proxy-name>.proxy-<random>.<region>.rds.amazonaws.com`[^rdsproxy]

---

## 3. ELB / ALB / NLB 負載均衡器

**所有負載均衡器（ALB、NLB、CLB、GWLB）共用同一格式：**

```
<name>-<generated-id>.elb.<region>.amazonaws.com
```

範例：`my-load-balancer-1234567890abcdef.elb.us-east-2.amazonaws.com`[^elbstd]

**每個可用區的節點 DNS 名稱：**

```
<az>.<name>-<generated-id>.elb.<region>.amazonaws.com
```

範例：`us-east-2b.my-load-balancer-1234567890abcdef.elb.us-east-2.amazonaws.com`[^elbaz]

- 名稱限制：最多 32 字元，僅字母數字與連字號，不可開頭或結尾為連字號，不可前綴 `internal-`，區域內唯一，建立後不可修改[^elbname]

---

## 4. AWS 服務端點

**標準區域端點：**

```
protocol://service-code.region-code.amazonaws.com
```

範例：`https://dynamodb.us-west-2.amazonaws.com`[^srvstd]

**FIPS 端點：** `service-code-fips.region-code.amazonaws.com`（如 `kms-fips.us-west-2.amazonaws.com`）[^fips]

**Dual-stack 端點（新格式）：**

```
protocol://service-code.region-code.api.aws
```

範例：`https://ec2.us-west-2.api.aws`[^dualstack]

> S3 例外：`protocol://service-code.dualstack.region-code.amazonaws.com`[^s3dual]

**通用端點（無區域，路由至 us-east-1）：** `ec2.amazonaws.com`、`autoscaling.amazonaws.com`、`elasticmapreduce.amazonaws.com`[^genep]

**全球端點：** CloudFront、IAM（`iam.amazonaws.com`）、Route 53（`route53.amazonaws.com`）、AWS Organizations、Global Accelerator、Shield Advanced、WAF Classic[^globalep]

**中國區域：** 使用 `amazonaws.com.cn` 域（`cn-north-1` 北京、`cn-northwest-1` 寧夏）[^cn]

**VPC 端點（PrivateLink）：**

```
vpce-<endpoint-id>-<hash>.<service-name>.<region>.vpce.amazonaws.com
```

範例：`vpce-099deb00b40f00e22-lj2wisx3.monitoring.us-east-2.vpce.amazonaws.com`[^vpce]

---

## 5. 其他服務

**S3：**
- 虛擬主機樣式：`<bucket>.s3.<region>.amazonaws.com`[^s3vh]
- 路徑樣式：`s3.<region>.amazonaws.com/<bucket>/<key>`[^s3path]
- 網站端點：`<bucket>.s3-website-<region>.amazonaws.com`（僅 HTTP）[^s3web]

**ElastiCache：** `<cluster>.<id>.<node>.<regionaz>.cache.amazonaws.com`[^cache]

**Redshift：** `<cluster>.<random-12>.<region>.redshift.amazonaws.com`（埠 5439）[^rs]

**MSK：** `b-<num>.<cluster>.<uuid>.c<version>.kafka.<region>.amazonaws.com`[^msk]

---

## 6. Route 53 命名限制

- 每個標籤最長 **63 bytes**，完整域名含點最長 **255 bytes**[^r53len]
- 主機託管區與記錄名稱可包含任何可列印 ASCII（不含空格）[^r53char]
- 註冊域名僅限 a-z、0-9、`-`，連字號不可在標籤開頭或結尾[^r53reg]
- 萬用字元 `*` 僅允許作為最左側標籤使用[^r53wild]
- 國際域名 (IDN) 以 Punycode 儲存（如 `xn--fiqs8s.asia` = 中國.asia）[^idn]

---

## 7. us-east-1 例外彙整

`us-east-1`（維吉尼亞北部）在 DNS 命名上有多項與其他區域不同之處：

| 面向 | us-east-1 | 其他區域 |
|---|---|---|
| 私有域 | `.ec2.internal` | `.<region>.compute.internal` |
| IP 主機名 | `ip-x-x-x-x.ec2.internal` | `ip-x-x-x-x.<region>.compute.internal` |
| 資源主機名 | `i-xxx.ec2.internal` | `i-xxx.<region>.compute.internal` |

## 參考文獻

[^pubdns-req]: Amazon Web Services. (n.d.). _DNS attributes for your VPC_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/vpc/latest/userguide/vpc-dns.html
[^pubipv4ex]: Amazon Web Services. (n.d.). _EC2 instance hostname types_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hostname-types.html
[^splitdns]: Amazon Web Services. (n.d.). _Understanding EC2 instance hostnames and domains_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/understanding-ec2-instance-hostnames-domains.html
[^ipv6ex]: Amazon Web Services. (n.d.). _EC2 instance hostname types_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hostname-types.html
[^dualex]: Amazon Web Services. (n.d.). _EC2 instance hostname types_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hostname-types.html
[^ipchange]: Amazon Web Services. (n.d.). _Understanding EC2 instance hostnames and domains_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/understanding-ec2-instance-hostnames-domains.html
[^useast1int]: Amazon Web Services. (n.d.). _EC2 instance hostname types_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hostname-types.html
[^othint]: Amazon Web Services. (n.d.). _EC2 instance hostname types_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hostname-types.html
[^rbnuseast]: Amazon Web Services. (n.d.). _EC2 instance hostname types_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hostname-types.html
[^rbnoth]: Amazon Web Services. (n.d.). _EC2 instance hostname types_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hostname-types.html
[^rbnipv6]: Amazon Web Services. (n.d.). _EC2 instance hostname types_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hostname-types.html
[^dnsaddr]: Amazon Web Services. (n.d.). _Amazon DNS concepts_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/vpc/latest/userguide/AmazonDNS-concepts.html
[^rdsstd]: Amazon Web Services. (n.d.). _Connecting to a DB instance_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.EndpointAndPort.html
[^rdsnaming]: Amazon Web Services. (n.d.). _Limits for Amazon RDS_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Limits.html
[^rdsfixid]: Amazon Web Services. (n.d.). _Connecting to a DB instance_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.EndpointAndPort.html
[^auroraw]: Amazon Web Services. (n.d.). _Aurora endpoints_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.Endpoints.html
[^aurorar]: Amazon Web Services. (n.d.). _Aurora reader endpoint_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Reader.html
[^aurorai]: Amazon Web Services. (n.d.). _Aurora instance endpoint_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Instance.html
[^aurorac]: Amazon Web Services. (n.d.). _Aurora custom endpoint_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Custom.html
[^rdsproxy]: Amazon Web Services. (n.d.). _RDS Proxy endpoints_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy-endpoints.html
[^elbstd]: Amazon Web Services. (n.d.). _Application Load Balancers_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancers.html
[^elbaz]: Amazon Web Services. (n.d.). _Network Load Balancers_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancers.html
[^elbname]: Amazon Web Services. (n.d.). _Application Load Balancers_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancers.html
[^srvstd]: Amazon Web Services. (n.d.). _AWS service endpoints_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/general/latest/gr/rande.html
[^fips]: Amazon Web Services. (n.d.). _AWS service endpoints_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/general/latest/gr/rande.html
[^dualstack]: Amazon Web Services. (n.d.). _AWS service endpoints_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/general/latest/gr/rande.html
[^s3dual]: Amazon Web Services. (n.d.). _Amazon S3 endpoints_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/general/latest/gr/s3.html
[^genep]: Amazon Web Services. (n.d.). _AWS service endpoints_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/general/latest/gr/rande.html
[^globalep]: Amazon Web Services. (n.d.). _AWS service endpoints_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/general/latest/gr/rande.html
[^cn]: Amazon Web Services. (n.d.). _Endpoints in the China Regions_. Retrieved 2026-09-20, from https://docs.amazonaws.cn/en_us/general/latest/gr/endpoints-Beijing.html
[^vpce]: Amazon Web Services. (n.d.). _Interface VPC endpoints_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/vpc/latest/privatelink/vpce-interface.html
[^s3vh]: Amazon Web Services. (n.d.). _Virtual hosting of buckets_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AmazonS3/latest/userguide/VirtualHosting.html
[^s3path]: Amazon Web Services. (n.d.). _Accessing a bucket_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-bucket-intro.html
[^s3web]: Amazon Web Services. (n.d.). _Amazon S3 endpoints_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/general/latest/gr/s3.html
[^cache]: Amazon Web Services. (n.d.). _ElastiCache endpoints_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/Endpoints.html
[^rs]: Amazon Web Services. (n.d.). _Connecting to a cluster_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/redshift/latest/mgmt/connecting-to-cluster.html
[^msk]: Amazon Web Services. (n.d.). _MSK client access_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/msk/latest/developerguide/client-access.html
[^r53len]: Amazon Web Services. (n.d.). _Domain name format for Route 53_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/DomainNameFormat.html
[^r53char]: Amazon Web Services. (n.d.). _Domain name format for Route 53_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/DomainNameFormat.html
[^r53reg]: Amazon Web Services. (n.d.). _Domain name format for Route 53_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/DomainNameFormat.html
[^r53wild]: Amazon Web Services. (n.d.). _Domain name format for Route 53_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/DomainNameFormat.html
[^idn]: Amazon Web Services. (n.d.). _Domain name format for Route 53_. Retrieved 2026-09-20, from https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/DomainNameFormat.html